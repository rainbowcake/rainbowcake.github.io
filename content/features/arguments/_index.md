+++
title = "Arguments"
weight = 20
+++

<div class="small-subtitle">rainbowcake-navigation</div>

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

### Optional arguments

Optional arguments can be handled very similarly to regular arguments.

Only a couple changes have to be made:

- The argument in the `newInstance` method needs to be optional (nullable, with a default value of `null`).
- The property also has to be nullable and `null` by default.
- Instead of the `requireXyz` functions, `getXyzOrNull` style functions are to be used.

An example of argument handling with both a required and an optional argument:

```kotlin
//region Arguments
@Suppress("ConvertSecondaryConstructorToPrimary")
@Deprecated(message = "Use newInstance instead", replaceWith = ReplaceWith("ArgExampleFragment.newInstance()"))
constructor()

companion object {
    private const val ARG_REQUIRED_ID = "ARG_REQUIRED_ID"
    private const val ARG_OPTIONAL_ID = "ARG_OPTIONAL_ID"

    @Suppress("DEPRECATION")
    fun newInstance(requiredId: Long, optionalId: Long? = null): ArgExampleFragment {
        return ArgExampleFragment().applyArgs {
            putLong(ARG_REQUIRED_ID, requiredId)
            if (optionalId != null) {
                putLong(ARG_OPTIONAL_ID, optionalId)
            }
        }
    }
}

private var requiredId: Long = 0
private var optionalId: Long? = null

private fun initArguments() {
    requiredId = requireArguments().requireLong(ARG_REQUIRED_ID)
    optionalId = requireArguments().getLongOrNull(ARG_OPTIONAL_ID)
}
//endregion
```
