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

## Fragment sample

###	The usual way -

```kotlin
class SampleFragment : Fragment() {

    private var _binding: FragmentSampleBinding? = null
    private val binding get() = _binding!!

    override fun onCreateView(
        inflater: LayoutInflater,
        container: ViewGroup?,
        savedInstanceState: Bundle?
    ): View {
        _binding = FragmentSampleBinding.inflate(inflater, container, false)
        return binding.root
    }

    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)

        with(binding) {
            textView.text = "Some text"
            button.setOnClickListener {
                //some action
            }
        }

    }

    override fun onDestroyView() {
        super.onDestroyView()
        _binding = null
    }

}
```
###	With the delegate

The delegate considers fragment's view lifecycle, clears the reference when current view is destroyed and prevent memory leaks.

```kotlin
class ListFragment : Fragment(R.layout.fragment_sample) {

    private val binding: FragmentSampleBinding by viewBindings()

    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)

        with(binding) {
            textView.text = "Some text"
            button.setOnClickListener {
                //some action
            }
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
