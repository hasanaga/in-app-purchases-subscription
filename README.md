# Welcome to in-app-purchase (Android Studio Java*)

Consumable Item In-App Purchases: https://github.com/wdtheprovider/in-app-purchase

In this repository i'm going to show you how to integrate In-App Purchase for Subscription of Google Play Billing version 4+ in 7 steps. I follow the officailly google 
 docs, i'm not using any third-party library

Demo <br>

<span> <img src="https://github.com/wdtheprovider/in-app-purchases-subscription/blob/master/app/src/main/res/drawable/s1.png" width="290" height="600">
 <span> <img src="https://github.com/wdtheprovider/in-app-purchases-subscription/blob/master/app/src/main/res/drawable/s2.png" width="290" height="600">
<span> <img src="https://github.com/wdtheprovider/in-app-purchases-subscription/blob/master/app/src/main/res/drawable/s3.png" width="290" height="600">
<span> <img src="https://github.com/wdtheprovider/in-app-purchases-subscription/blob/master/app/src/main/res/drawable/s4.png" width="290" height="600">
<span> <img src="https://github.com/wdtheprovider/in-app-purchases-subscription/blob/master/app/src/main/res/drawable/s5.png" width="290" height="600">
<span> <img src="https://github.com/wdtheprovider/in-app-purchases-subscription/blob/master/app/src/main/res/drawable/s6.png" width="290" height="600">

</span>

<br>

[![Donate](https://img.shields.io/badge/Donate-PayPal-green.svg)](https://www.paypal.com/donate/?hosted_button_id=KPSJWR36UHBV2)

<br>

Pre-requisite
- Google Play Console Account
- Published App on Play Store
- Tester Device with GMS

YouTube Video: Part-1 | Intro Demo: Uploading soon <br>
<br>YouTube Video: Part-2 | Configure Testing Device: https://youtu.be/j6wWVMj-fi8 <br>
<br>YouTube Video: Part-3 | Integrating The Methods to purchase the products: Uploading soon <br>

```
Configure Your Testing device by adding the gmail account to internal testing testers 
and License testing (Watch the YouTube video for clarity: https://youtu.be/j6wWVMj-fi8 )


Setup the in-app purchase subscription product in Google Play Console account
i have already created mine which are 
Product ID: sub_premium

```

The following methods (These are the methods you need for the IAP System to work, you can copy and paste)

```java
void establishConnection(){}
void showProducts(){}
void launchPurchaseFlow(){}
void verifySubPayment(Purchase purchases){}
void checkSubscription(){}
```

[**Step 1: Add the Google Play Billing Library dependency**](#step-1-add-the-google-play-billing-library-dependency)

[**Step 2: Initialize a BillingClient with PurchasesUpdatedListener**](#step-2-initialize-a-billingclient-with-purchasesupdatedlistener)

[**Step 3: Establish a connection to Google Play**](#step-3-establish-a-connection-to-google-play)

[**Step 4: Show products available to buy**](#step-4-show-products-available-to-buy)

[**Step 5: Launch the purchase flow**](#step-5-launch-the-purchase-flow)

[**Step 6: Processing purchases / Verify Payment**](#step-6-processing-purchases--verify-payment)

[**Step 7: Handling pending transactions**](#step-7-handling-pending-transactions)

[**Step 8: Check the subscriptions on SplashScreenActivity**](#step-8-check-subscription-this-code-goes-to-your-splash-screen-)

<br> Learn More: https://developer.android.com/google/play/billing/integrate

### Step 1: Add the Google Play Billing Library dependency<br>
```gradle
//Add the Google Play Billing Library dependency to your app's build.gradle file as shown:

dependencies {
    def billingVersion = "5.0.0"

    implementation "com.android.billingclient:billing:$billingVersion"
}
```

```xml
//And Open Manifest File and add this permission
<uses-permission android:name="com.android.vending.BILLING" />

```
### Step 2: Initialize a BillingClient with PurchasesUpdatedListener<br>

```java
  //Initialize a BillingClient with PurchasesUpdatedListener onCreate method

    billingClient = BillingClient.newBuilder(this)
                .enablePendingPurchases()
                .setListener(
                        new PurchasesUpdatedListener() {
                            @Override
                            public void onPurchasesUpdated(@NonNull BillingResult billingResult, @Nullable List<Purchase> list) {
                               if(billingResult.getResponseCode()==BillingClient.BillingResponseCode.OK && list !=null) {
                                   for (Purchase purchase: list){
                                       verifySubPurchase(purchase);
                                   }
                               }
                            }
                        }
                ).build();

        //start the connection after initializing the billing client
        establishConnection();
                
```
### Step 3: Establish a connection to Google Play<br>

```java
 void establishConnection() {

        billingClient.startConnection(new BillingClientStateListener() {
            @Override
            public void onBillingSetupFinished(@NonNull BillingResult billingResult) {
                if (billingResult.getResponseCode() == BillingClient.BillingResponseCode.OK) {
                    // The BillingClient is ready. You can query purchases here.
                    showProducts();
                }
            }

            @Override
            public void onBillingServiceDisconnected() {
                // Try to restart the connection on the next request to
                // Google Play by calling the startConnection() method.
                establishConnection();
            }
        });
    }

    
```
### Step 4: Show products available to buy<br>

```java
@SuppressLint("SetTextI18n")
    void showProducts() {

        ImmutableList<QueryProductDetailsParams.Product> productList = ImmutableList.of(
                //Product 1 = index is 0
                QueryProductDetailsParams.Product.newBuilder()
                .setProductId("sub_premium")
                .setProductType(BillingClient.ProductType.SUBS)
                .build(),

                //Product 2 = index is 1
                QueryProductDetailsParams.Product.newBuilder()
                .setProductId("test_id_shar")
                .setProductType(BillingClient.ProductType.SUBS)
                .build()

        );

        QueryProductDetailsParams params = QueryProductDetailsParams.newBuilder()
                .setProductList(productList)
                .build();

        billingClient.queryProductDetailsAsync(
                params,
                (billingResult, productDetailsList) -> {
                    // Process the result
                    for (ProductDetails productDetails : productDetailsList) {
                        if (productDetails.getProductId().equals("sub_premium")) {
                            List<ProductDetails.SubscriptionOfferDetails> subDetails = productDetails.getSubscriptionOfferDetails();
                            assert subDetails != null;
                            Log.d("testOffer",subDetails.get(0).getOfferToken());
                            txt_price.setText(subDetails.get(0).getPricingPhases().getPricingPhaseList().get(0).getFormattedPrice()+" Per Month");
                            txt_price.setOnClickListener(view -> {
                                launchPurchaseFlow(productDetails);
                            });
                        }

                        if (productDetails.getProductId().equals("test_id_shar")) {
                            List<ProductDetails.SubscriptionOfferDetails> subDetails = productDetails.getSubscriptionOfferDetails();
                            assert subDetails != null;
                            Log.d("testOffer",subDetails.get(1).getOfferToken());
                            offer_btn.setText(subDetails.get(1).getPricingPhases().getPricingPhaseList().get(0).getFormattedPrice()+" Per Month");
                            offer_btn.setOnClickListener(view -> {
                                launchPurchaseFlow(productDetails);
                            });
                        }
                    }
                }
        );

    }
    
```
### Step 5: Launch the purchase flow<br>

```java
    void launchPurchaseFlow(ProductDetails productDetails) {
        assert productDetails.getSubscriptionOfferDetails() != null;
        ImmutableList<BillingFlowParams.ProductDetailsParams> productDetailsParamsList =
                ImmutableList.of(
                        BillingFlowParams.ProductDetailsParams.newBuilder()
                                .setProductDetails(productDetails)
                                .setOfferToken(productDetails.getSubscriptionOfferDetails().get(0).getOfferToken())
                                .build()
                );
        BillingFlowParams billingFlowParams = BillingFlowParams.newBuilder()
                .setProductDetailsParamsList(productDetailsParamsList)
                .build();
        BillingResult billingResult = billingClient.launchBillingFlow(activity, billingFlowParams);
    }
    
```
### Step 6: Processing purchases / Verify Payment<br>

```java
 
    void verifySubPurchase(Purchase purchases) {

        AcknowledgePurchaseParams acknowledgePurchaseParams = AcknowledgePurchaseParams
                .newBuilder()
                .setPurchaseToken(purchases.getPurchaseToken())
                .build();

        billingClient.acknowledgePurchase(acknowledgePurchaseParams, billingResult -> {
            if (billingResult.getResponseCode() == BillingClient.BillingResponseCode.OK) {
                //user prefs to set premium
                Toast.makeText(StoreActivity.this, "You are a premium user now", Toast.LENGTH_SHORT).show();
                //Setting premium to 1
                // 1 - premium
                // 0 - no premium
                prefs.setPremium(1);
            }
        });

        Log.d(TAG, "Purchase Token: " + purchases.getPurchaseToken());
        Log.d(TAG, "Purchase Time: " + purchases.getPurchaseTime());
        Log.d(TAG, "Purchase OrderID: " + purchases.getOrderId());
    }
    
```

### Step 7: Handling pending transactions<br>

```java
   
    protected void onResume() {
        super.onResume();
        billingClient.queryPurchasesAsync(
                QueryPurchasesParams.newBuilder().setProductType(BillingClient.ProductType.SUBS).build(),
                (billingResult, list) -> {
                    if (billingResult.getResponseCode() == BillingClient.BillingResponseCode.OK) {
                        for (Purchase purchase : list) {
                            if (purchase.getPurchaseState() == Purchase.PurchaseState.PURCHASED && !purchase.isAcknowledged()) {
                                verifySubPurchase(purchase);
                            }
                        }
                    }
                }
        );

    }
```


### Step 8: Check Subscription (This code goes to your Splash screen) <br>

```java
void checkSubscription(){

        billingClient = BillingClient.newBuilder(this).enablePendingPurchases().setListener((billingResult, list) -> {}).build();
        final BillingClient finalBillingClient = billingClient;
        billingClient.startConnection(new BillingClientStateListener() {
            @Override
            public void onBillingServiceDisconnected() {

            }

            @Override
            public void onBillingSetupFinished(@NonNull BillingResult billingResult) {

                if(billingResult.getResponseCode() == BillingClient.BillingResponseCode.OK){
                    finalBillingClient.queryPurchasesAsync(
                            QueryPurchasesParams.newBuilder().setProductType(BillingClient.ProductType.SUBS).build(), (billingResult1, list) -> {
                        if (billingResult1.getResponseCode() == BillingClient.BillingResponseCode.OK){
                             Log.d("testOffer",list.size() +" size");
                             if(list.size()>0){
                                 prefs.setPremium(1); // set 1 to activate premium feature
                                 int i = 0;
                                 for (Purchase purchase: list){
                                     //Here you can manage each product, if you have multiple subscription
                                     Log.d("testOffer",purchase.getOriginalJson()); // Get to see the order information
                                     Log.d("testOffer", " index" + i);
                                     i++;
                                 }
                             }else {
                                 prefs.setPremium(0); // set 0 to de-activate premium feature
                             }
                        }
                    });

                }

            }
        });
    }
 
```




        

