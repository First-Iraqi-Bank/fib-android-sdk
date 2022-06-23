## Feature:

FIB Payment SDK was created to allow you to integrate FIB as a convenient payment method within your application.

* Initiate a transaction and let users pay through FIB apps (**personal**, **business**, **corporate**)
* Get payment status
* Cancel payments

## Requirements:

To use this SDK on Android, the following must be addressed.

* API level 21+.
* Have a developer account, and provide the SDK with your account credentials such as ClientID, ClientSecret, GrantType
  and FIBBackend.

## Download:

### Gradle:

Add this dependency in your app’s `build.gradle` file

```groovy
dependencies {
    implementation 'iq.fib.payments:fib_payment_sdk:1.0.0'
}
```

## Initializing SDK:

### Step 1: Get your account credentials:

After you took required steps to get a developer account in FIB, you'll be provided with your accounts credentials (
FIBBackend, GrantType, ClientId, ClientSecret)

### Step 2: Provide your credentials to the SDK:

In the root of your android project locate `local.properties`(create one if it doesn't’t exist) file and add your
credentials to it.

```groovy
FIBBackend = XXXX
GrantType = XXXX
ClientId = XXXX
ClientSecret = XXXX
```

### Step 3: Provide your environment to the SDK:

In your app’s `build.gradle` file add `it.manifestPlaceholders["env_type"] = ".dev"` to your `defaultConfig` block

```groovy
android {
    compileSdk 30
    defaultConfig {
        // replace ".dev" with "" for production environment
        //.dev: means that all transactions are fake and the operations will happen in a development environment 
        //"": means that the transactions are real and the operations will happen in a production environment
      it.manifestPlaceholders["env_type"] = ".dev"
    }
}
```

## Usage:

the SDK could be used in two ways:

* [By reusing provided UI components](#using-provided-ui)
* [By using custom UI and reusing the underlying logic](#using-provided-logic-with-custom-ui-components)

### Using provided UI:

You can use `FIBButton` view to do your payment operations

Add `FIBButton` to your layout class:

```xml

<com.lawrencespring.payment.widget.FIBButton
        android:id="@+id/buttonBuy"
        android:layout_width="0dp"
        android:layout_height="wrap_content"
        app:layout_constraintLeft_toLeftOf="parent"
        app:layout_constraintRight_toRightOf="parent"
        app:layout_constraintBottom_toBottomOf="parent"/>
```

reference the button in your activity/fragment code

```kotlin
import com.lawrencespring.payment.widget.FIBButton

val payButton = findViewById<FIBButton>(R.id.buttonBuy)
```

payment operation with `FIBButton`

```kotlin
  payButton.setOnClickListener {
    //starting payment with FIBButton
    payButton.startPayment(
        listOf(
            "https://personal.dev.first-iraqi-bank.co/?link=https://personal.dev.first-iraqi-bank.co/onlinePayment/?identifier%3DSIBAFZSPVZB2&apn=com.firstiraqibank.personal.dev&ibi=se.core.Lawrencespring.development",
            "https://business.dev.first-iraqi-bank.co/?link=https://business.dev.first-iraqi-bank.co/onlinePayment/?identifier%3DSIBAFZSPVZB2&apn=com.firstiraqibank.business.dev&ibi=se.core.Lawrencespring.business.development",
            "https://corporate.dev.first-iraqi-bank.co/?link=https://corporate.dev.first-iraqi-bank.co/onlinePayment/?identifier%3DSIBAFZSPVZB2&apn=com.firstiraqibank.corporate.dev&ibi=se.core.Lawrencespring.corporate.development"
        )
    )
}

//retrieving payment status
lifecycleScope.launch {
    val state: PaymentStatus? = payButton.checkPaymentState(paymentID)
}

//canceling payment 
lifecycleScope.launch {
    payButton.cancelPayment(paymentID)
}

```

### Using provided Logic with custom UI components

payment operation with `PayWithFIB`

```kotlin
import com.lawrencespring.payment.adapter.exposer.payWithFIB.PayWithFIB
import com.lawrencespring.payment.framework.exposer.payWithFIB.PaymentListener
import com.lawrencespring.payment.framework.exposer.payWithFIB.model.FIBApp
import com.lawrencespring.payment.framework.exposer.payWithFIB.PaymentStatus

val payWithFIB = PayWithFIB()

//moreThanOneApp is triggered only when multiple FIB apps are installed on the end device
//once the app is selected it could be started by using appStarter(FIBApp)
payWithFIB.addPaymentListener(object : PaymentListener {
    //availableApps: are the list of fib apps available on the end device
    //appStarted: is a callback function that's called with the app that needs to be open     
    override fun moreThanOneApp(availableApps: List<FIBApp>, appStarter: (FIBApp) -> Unit) {
        //creating a dialog that will show the available apps
        val dialog = AlertDialog.Builder(context)
        val arrayAdapter =
            ArrayAdapter<String>(context, android.R.layout.select_dialog_item)

        availableApps.forEach { arrayAdapter.add(it.name.lowercase()) }
        dialog.setAdapter(arrayAdapter) { _, which ->
            //passing the selected app to the callback          
            appStarter(availableApps[which])
        }

        dialog.create().show()
    }
})

//starting payment with PayWithFIB
payWithFIB.startPayment(
    listOf(
        "https://personal.dev.first-iraqi-bank.co/?link=https://personal.dev.first-iraqi-bank.co/onlinePayment/?identifier%3DSIBAFZSPVZB2&apn=com.firstiraqibank.personal.dev&ibi=se.core.Lawrencespring.development",
        "https://business.dev.first-iraqi-bank.co/?link=https://business.dev.first-iraqi-bank.co/onlinePayment/?identifier%3DSIBAFZSPVZB2&apn=com.firstiraqibank.business.dev&ibi=se.core.Lawrencespring.business.development",
        "https://corporate.dev.first-iraqi-bank.co/?link=https://corporate.dev.first-iraqi-bank.co/onlinePayment/?identifier%3DSIBAFZSPVZB2&apn=com.firstiraqibank.corporate.dev&ibi=se.core.Lawrencespring.corporate.development"
    )
)


//retrieving payment status
lifecycleScope.launch {
    val state: PaymentStatus? = payWithFIB.getPaymentState(paymentID)
}

//canceling payment 
lifecycleScope.launch {
    payWithFIB.cancelPayment(paymentID)
}

```

The library will check if any version of FIB app is installed on the mobile device of the user and will try to proceed
transaction. If more than one app are installed the user is asked to choose one of them. If no FIB is installed the user
will be redirected to the Google Play Store.

License
-------

```
Copyright 2021 first iraqi bank, Inc.

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

   https://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
```
## Redirection

this SDK also includes an option to redirect the user from the FIB appllications to your application, you can provide the applications `redirect URI` and everyThing will be handled for you.

it is optional to have this feature, so if you provide your `redirect URI`, the redirection happens otherwise the FIB applications behave as they normally would.
