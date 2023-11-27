---
cover: ../../.gitbook/assets/Blue Modern Marketing Manager LinkedIn Banner.png
coverY: 0
---

# Customers API

Our customers API allows you to securely save customer data and payment methods. In order to associate customer's payment method for future payments you have to provide customer UUID in the transaction params, details below.

### Your backend

In order to operate with our Customer API you have to obtain access token and forward to your mobile app.

Here is example how to create access token with your backend:

POST request on endpoint https://ipgtest.monri.com/v2/oauth Body:

```json
{
  "client_id": "Authenticity token",
  "client_secret": "Merchant key",
  "scopes": [
    "customers",
    "payment-methods"
  ],
  "grant_type": "client_credentials"
}
```

Response example:

```json
{
  "access_token": "*********************",
  "token_type": "Bearer",
  "expires_in": 900,
  "status": "approved"
}
```

### Activity setup

Create Monri instance in onCreate method (or before):

```java
@Override
protected void onCreate(Bundle savedInstanceState) {
    //...
        monri = new Monri(((ActivityResultCaller) this), MonriApiOptions.create("merchant authenticity token", true));
    //...
}
```

### Create a customer

In order to create a `Customer` you’ve to provide:

* access token
* `CustomerData`

```java
//...
String accessToken = "Bearer *********************" //access token from backend

CustomerData customerData = new CustomerData()
.setMerchantCustomerUuid(merchantCustomerId)//optional, you can use your own uuid
.setDescription("description")
.setEmail("harun.kolos@monri.com")
.setName("Harun")
.setPhone("00387000111")
.setMetadata(new HashMap<>() {{
    put("a", "b");
}})
.setZipCode("71000")
.setCity("Sarajevo")
.setAddress("Džemala Bijedića 2")
.setCountry("BA");
                
CreateCustomerParams createCustomerParams = new CreateCustomerParams(
                customerData,
                accessToken
        );

monri.getMonriApi().customers().create(
createCustomerParams,
new ResultCallback<Customer>() {
    @Override
    public void onSuccess(final Customer result) {
      //...
    }

    @Override
    public void onError(final Throwable throwable) {
      //...
    }
});
//...
```

### Update a customer

In order to update a `Customer` you’ve to provide:

* access token
* `UpdateCustomerParams`

```java
//...
final CustomerData updateCustomerData = new CustomerData()
                                .setEmail("update@email.com")
                                .setName("Harun")
                                .setPhone("00387000112");
                        
final UpdateCustomerParams updateCustomerParams = new UpdateCustomerParams(
                                updateCustomerData,
                                "created customer uuid",
                                "accessToken from backend"
                        );
monri.getMonriApi()
     .customers()
     .update(
              updateCustomerParams,
              new ResultCallback<Customer>() {
                  @Override
                  public void onSuccess(final Customer result) {
                      //...
                  }
  
                  @Override
                  public void onError(final Throwable throwable) {
                      //...
                  }
              }
            );
//...
```

### Delete a customer

In order to delete a `Customer` you’ve to provide:

* access token
* `DeleteCustomerParams`

```java
//...
final DeleteCustomerParams deleteCustomerParams = new DeleteCustomerParams(
                                "accessToken",
                                "created customer uuid"
                        );
monri.getMonriApi().customers().delete(
        deleteCustomerParams,
        new ResultCallback<DeleteCustomerResponse>() {
            @Override
            public void onSuccess(final DeleteCustomerResponse result) {
               //...
            }

            @Override
            public void onError(final Throwable throwable) {
                //..
            }
        }
);
//...
```

### Retrieve a customer

In order to retrieve a `Customer` you’ve to provide:

* access token
* `GetCustomerParams`

```java
//...
final GetCustomerParams retrieveCustomerParams = new GetCustomerParams(
                                "accessToken",
                                "created customer uuid"
                        );
monri.getMonriApi().customers().get(
        retrieveCustomerParams,
        new ResultCallback<Customer>() {
            @Override
            public void onSuccess(final Customer result) {
                //...
            }

            @Override
            public void onError(final Throwable throwable) {
                //...
            }
        }
);
//...
```

### Retrieve a customer via Merchant Uuid

In order to retrieve a `Customer` via Merchant Uuid you’ve to provide:

* access token
* `RetrieveCustomerViaMerchantCustomerUuidParams`

```java
//...
final RetrieveCustomerViaMerchantCustomerUuidParams retrieveCustomerViaMerchantCustomerUuidParams = new RetrieveCustomerViaMerchantCustomerUuidParams(
                                "accessToken",
                                "created customer uuid"
                        );
monri.getMonriApi().customers().getViaMerchantCustomerUuid(
        retrieveCustomerViaMerchantCustomerUuidParams,
        new ResultCallback<Customer>() {
            @Override
            public void onSuccess(final Customer result) {
                //...
            }

            @Override
            public void onError(final Throwable throwable) {
                //...
            }
        }
);
//...
```

### Retrieve all customers

In order to get all customers you’ve to provide:

* access token

```java
//...
monri.getMonriApi().customers().all("accessToken", new ResultCallback<MerchantCustomers>() {
    @Override
    public void onSuccess(final MerchantCustomers result) {
        //...
    }

    @Override
    public void onError(final Throwable throwable) {
        //...
    }
});
//...
```

### Retrieve customer payment methods

In order to retrieve all customer payment methods you’ve to provide:

* access token
* `CustomerPaymentMethodParams`

```java
//...
final CustomerPaymentMethodParams customerPaymentMethodParams = new CustomerPaymentMethodParams(
                                "created customer uuid",
                                20,//limit
                                0,//offset
                                "accessToken"
                        );
monri.getMonriApi().customers().paymentMethods(
        customerPaymentMethodParams,
        new ResultCallback<CustomerPaymentMethodResponse>() {
            @Override
            public void onSuccess(final CustomerPaymentMethodResponse result) {
                //...
            }

            @Override
            public void onError(final Throwable throwable) {
                //...
            }
        }
);
//...
```

### Confirm payment with customer UUID - save card for future payments

In order to associate customer's payment method for future payments, beside customer's data which is optional, you have to provide also:

* created customer UUID
* clientSecret - please see our section about [Payment API Integration](payment-api-integration.md)

```java
final Card card = new Card("4111 1111 1111 1111", 12, 2034, "123");

final CustomerParams customerParams = new CustomerParams()
                .setCustomerUuid("created customer UUID")
                .setAddress("Adresa")
                .setFullName("Harun Kolos")
                .setCity("Sarajevo")
                .setZip("71000")
                .setPhone("+38761000111")
                .setCountry("BA")
                .setEmail("monri-android-sdk-test@monri.com");

card.setTokenizePan(true);//save card for future payment

ConfirmPaymentParams confirmPaymentParams = ConfirmPaymentParams.create(
        "clientSecret",
        card.toPaymentMethodParams(),
        TransactionParams.create()
                .set("order_info", "Android SDK payment session")
                .set(customerParams)
);
        
monri.confirmPayment(confirmPaymentParams, (PaymentResult result, Throwable cause) -> {
    if (cause == null) {
        //... handle result
    } else {
        //... handle error
    }
});
```
