# ViewBindingDelegate

A delegate that makes it a little easier to work with [Android View Binding](https://d.android.com/topic/libraries/view-binding).  

To enable view binding, set the viewBinding build option to true in the module-level build.gradle file, as shown in the following example:

```groovy
android {
    buildFeatures {
        viewBinding true
    }
}
```
Please note that you need to copy the source code to work with the delegate, because it's not currently in some repository.

## Delegate Fragment 

```kotlin
inline fun <reified B : ViewBinding> Fragment.viewBinding(): ViewBindingDelegate<B> {
    val fragment = this
    return ViewBindingDelegate(fragment, B::class.java)
}

class ViewBindingDelegate<B : ViewBinding>(
    private val fragment: Fragment,
    private val viewBindingClass: Class<B>
) {

    var binding: B? = null

    operator fun getValue(thisRef: Any?, property: KProperty<*>): B {
        val viewLifecycleOwner = fragment.viewLifecycleOwner
        if (viewLifecycleOwner.lifecycle.currentState == Lifecycle.State.DESTROYED) {
            throw IllegalStateException("Called after onDestroyView()")
        } else if (fragment.view != null) {
            return getOrCreateBinding(viewLifecycleOwner)
        } else {
            throw IllegalStateException("Called before onViewCreated()")
        }
    }

    @Suppress("UNCHECKED_CAST")
    private fun getOrCreateBinding(lifecycleOwner: LifecycleOwner): B {
        return this.binding ?: let {
            val method = viewBindingClass.getMethod("bind", View::class.java)
            val binding = method.invoke(null, fragment.view) as B

            lifecycleOwner.lifecycle.addObserver(object : DefaultLifecycleObserver {
                override fun onDestroy(owner: LifecycleOwner) {
                    super.onDestroy(owner)
                    this@ViewBindingDelegate.binding = null
                }
            })

            this.binding = binding
            binding
        }
    }
}
```
###	With the delegate

The delegate considers fragment's view lifecycle, clears the reference when current view is destroyed and prevent memory leaks.

Usage example:
 
  ```kotlin
  class MyFragment : Fragment(R.layout.fragment_test) {
 
      private val binding by viewBinding<FragmentTestBinding>()
 
      fun doSomething() {
          binding.textView.text = "Oooops"
      }
  }
 ```

## Activity sample

```kotlin
class MainActivity : AppCompatActivity() {

    private val binding: ActivityMainBinding by viewBindings()

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(binding.root)
    }
}
```
