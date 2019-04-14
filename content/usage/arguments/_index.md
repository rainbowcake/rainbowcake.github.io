+++
title = "Arguments"
weight = 20
+++

Each screen should take relatively few and relatively simple arguments. This should usually be an ID of some sort, sometimes a boolean flag. The arguments will be passed in the Fragment's argument Bundle, which means they'll survive even process death (and do so automatically). Model objects should never be passed as arguments.

Every screen should be able to initialize itself from just these arguments alone, by pulling the data they have to actually display from the data sources (through the various layers). 

Fragments without arguments can simply be instantiated via their constructors, but Fragments with arguments will use factory methods. A method called `initArguments` by convention will read all argument Bundle arguments after the view has been created, but before it is initialized.

```kotlin
class PurchaseFragment : RainbowCakeFragment<PurchaseViewState, PurchaseViewModel> {

    // A public no-arg constructor to be only used by the framework
    // Clients will be redirected to use the `newInstance` method instead via the deprecation notice
    @Suppress("ConvertSecondaryConstructorToPrimary")
    @Deprecated(message = "Use newInstance instead", replaceWith = ReplaceWith("PurchaseFragment.newInstance()"))
    constructor()

    companion object {
        // A key for the argument
        private const val ARG_TRANSACTION_ID = "ARG_TRANSACTION_ID"

        // The factory method to instantiate the Fragment with
        @Suppress("DEPRECATION")
        fun newInstance(transactionId: String): PurchaseFragment {
            return PurchaseFragment().applyArgs { // applyArgs is a simple extension to create a Bundle quicker
                putString(ARG_TRANSACTION_ID, transactionId)
            }
        }
    }

    // The property where we can read the argument value from
    private lateinit var transactionId: String

    // A method that initializes all arguments from the Bundle
    private fun initArguments() {
        transactionId = requireArguments().requireString(ARG_TRANSACTION_ID)
    }

    // ... onCreateView, etc methods ...
    
    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)

        initArguments() // argument initialization
        
        setupList() // UI setup
        setupButtons()
    }
    
    override fun onStart() {
        super.onStart()
        
        viewModel.loadPurchaseData(transactionId) // Use the argument to fetch actual data
    }
}
```

See also: [required argument handling](/content/views/required-arguments.md) and [optional argument handling](/content/views/optional-arguments.md).
