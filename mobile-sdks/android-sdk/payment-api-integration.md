---
cover: ../../.gitbook/assets/cover.png
coverY: 0
---

# Payment API Integration

Our Android libraries let you easily accept mobile payments inside any Android app.

Monri has created a Java library for Android, allowing you to easily submit payments from an Android app. With our
mobile library, we address PCI compliance by eliminating the need to send card data directly to your server. Instead,
our libraries send the card data directly to our servers, where we can convert them to tokens.

Recently we've added a new way of simplified payment integration in your app.

It consists of two steps:

* create new payment on merchant's backend
* confirm created payment on merchant's mobile application using SDK

New integration is designed to guide customer through payment process in app.

It reduces integration time significantly simply by requiring implementation of two endpoints on merchant's backend.

Continue following this wiki to learn how to create payment on backend.

<figure><img src="../../.gitbook/assets/monri-android.png" alt="" width="375"><figcaption></figcaption></figure>

### Collecting credit card information

At some point in the flow of your app, you’ll want to obtain payment details from the user. There are a couple ways to
do this:

* Use our built-in card input widget to collect card information
* Build your own credit card form

Instructions for each route follows, although you may want to write your app to offer support for both.

### Using the card input widget

To collect card data from your customers directly, you can use
Monri’s [CardMultilineWidget](https://github.com/monri/monri-android/blob/master/monri/src/main/java/com/monri/android/view/CardMultilineWidget.java)
in your application. You can include it in any view’s layout file.

<figure><img src="../../.gitbook/assets/monri-android-cardmultiline.png" alt="" width="375"><figcaption></figcaption></figure>

```java
<com.monri.android.view.CardMultilineWidget
    android:id="@+id/card_input_widget"
    android:layout_width="match_parent"
    android:layout_height="wrap_content" />
```

This allows your customers to input all of the required data for their card: the number, the expiration date, and the
CVV code. Note that the value of the `Card` object is `null` if the data in the widget is either incomplete or fails
client-side validity checks.

```java
import com.monri.android.view.CardMultilineWidget;
CardMultilineWidget cardInputWidget = (CardMultilineWidget) findViewById(R.id.card_input_widget);

Card cardToSave = cardInputWidget.getCard();
if (cardToSave == null) {
    errorDialogHandler.showError("Invalid Card Data");
}
```

If you have any other data that you would like to associate with the card, such as name, address, or ZIP code, you can
put additional input controls on your layout and add them directly to the `Card` object.

```java
cardToSave = cardToSave.toBuilder().name("Customer Name").build();
cardToSave = cardToSave.toBuilder().addressZip("12345").build();
```

### Building your own form

If you build your own payment form, you’ll need to collect at least your customers’ card numbers and expiration dates.
Monri strongly recommends collecting the CVC. You can optionally collect the user’s name and billing address for
additional fraud protection.

Once you’ve collected a customer’s information, you will need to exchange the information for a Monri token.

#### Creating & validating cards from a custom form

To create a `Card` object from data you’ve collected from other forms, you can create the object with its constructor.

```java
import com.monri.android.model.Card;

//...
//...
public void onAddCard(String cardNumber, String cardExpMonth,
                      String cardExpYear, String cardCVC) {
  final Card card = Card.create(
    cardNumber,
    cardExpMonth,
    cardExpYear,
    cardCVC
  );

  card.validateNumber();
  card.validateCVC();
}
```

As you can see in the example above, the `Card` instance contains some helpers to validate that the card number passes
the Luhn check, that the expiration date is the future, and that the CVC looks valid. You’ll probably want to validate
these three things at once, so we’ve included a `validateCard` function that does so.

```
// The Card class will normalize the card number
final Card card = Card.create("4242-4242-4242-4242", 12, 2020, "123");
if (!card.validateCard()) {
  // Show errors
}
```

### Creating new payment on merchant's backend

This step is preferably executed when you have enough information to create customer's order.

For simplicity we'll show example using [curl](https://curl.haxx.se/) in PHP.

To create payment on our backend you'll need:

* `merchant_key` (available on merchant's dashboard)
* `authenticity_token` (available on merchant's dashboard)

Additionally we require following fields:

| field             | length | type    | description                                             |
|-------------------|--------|---------|---------------------------------------------------------|
| amount            | 1-11   | Integer | amount is in minor units, ie. 10.24 USD is sent as 1024 |
| order\_number     | 2-40   | String  | unique order identifier                                 |
| currency          | 3      | String  | One of supported currencies (BAM, EUR, USD, CHF etc)    |
| transaction\_type | enum   | String  | possible values are: `authorize` or `purchase`          |
| order\_info       | 3-100  | String  | short description of order being processed              |

Optionally we offer setting payment scenario, which can be one of:

| field    | length | type   | description                                           |
|----------|--------|--------|-------------------------------------------------------|
| scenario | enum   | String | possible values are: `charge` or `add_payment_method` |

Scenario `charge` charges customer amount. Depending on `transaction_type` amount is reserved (authorize) or captured (
purchase).

Scenario `add_payment_method` provides simple way to implement 'Save card for future payments' functionality.

For request authentication we use `Authorization` header created from:

* authorization schema: String = `WP3-v2`
* authenticity\_token: String = value from merchant's configuration
* timestamp: Integer = unix timestamp (eg PHP's time())
* digest: String = sha512(merchant\_key + timestamp + authenticity\_token + body\_as\_string)

Parts above are joined by space, so `Authorization` header should be in this form:

`Authorization: schema authenticity_token timestamp digest`

Example: `Authorization: WP3-v2 abc...def 1585229134 314d32d1...0b49`

Request endpoint is `<base_url>/v2/payment/new` where base\_url is:

* `https://ipgtest.monri.com` for TEST environment
* `https://ipg.monri.com` for PROD environment

_TIP_: Parametrize merchant\_key, authenticity\_token and base\_url so it can be easily changed when you are ready for
production environment.

Payment/new response contains:

* status: String: approved | declined
* id: String - Unique payment identifier used to track payment flow on Monri's side. Useful for debugging if something
  goes wrong. Save this value in your database.
* client\_secret: String - Value you'll send to your application which then will use this secret to confirm payment
  using Monri Android SDK.

Request example in PHP:

```php
$data = [
  "amount" => 100,
  // unique order identifier
  "order_number" => 'random'. time(),
  "currency" => "EUR",
  "transaction_type" => "purchase",
  "order_info" => "Create payment session order info",
  "scenario" => 'charge'
];
$body_as_string = Json::encode($data); // use php's standard library equivalent if Json::encode is not available in your code
$base_url = 'https://ipgtest.monri.com'; // parametrize this value
$ch = curl_init($base_url . '/v2/payment/new');
curl_setopt($ch, CURLOPT_CUSTOMREQUEST, "POST");
curl_setopt($ch, CURLOPT_POSTFIELDS, $body_as_string);
curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);
curl_setopt($ch, CURLOPT_SSL_VERIFYPEER, 1);
curl_setopt($ch, CURLOPT_SSL_VERIFYHOST, 0);

$timestamp = time();
$digest = hash('sha512', $key . $timestamp .$authenticity_token. $body_as_string);
$authorization = "WP3-v2 $authenticity_token $timestamp $digest";
            
curl_setopt($ch, CURLOPT_HTTPHEADER, array(
    'Content-Type: application/json',
    'Content-Length: ' . strlen($body_as_string),
    'Authorization: '.$authorization
  )
);

$result = curl_exec($ch);

if (curl_errno($ch)) {
  curl_close($ch);
  $response = ['client_secret' => null, 'status' => 'declined', 'error' => curl_error($ch)];
} else {
  curl_close($ch);
  $response = ['status' => 'approved', 'client_secret' => Json::decode($result)['client_secret']];
}

var_dump($response);
```

### Confirm payment on merchant's application

After you've created payment on a backend and sent client\_secret back to your application you need to confirm payment
using Monri's SDK.

Steps:

* ensure you have valid client\_secret (created on backend using payment/new)
* create Monri instance - this.monri = new Monri(context, monriApiOptions);
* implement onActivityResult

```java
@Override
protected void onActivityResult(int requestCode, int resultCode, @Nullable Intent data) {
  // For this to work you'll need to implement interface ResultCallback<PaymentResult>
  // onSuccess method is invoked if payment is successfully processed
  // onError method is invoked if an error occurred during processing
  ResultCallback<PaymentResult> callback = this;
  final boolean monriPaymentResult = monri.onPaymentResult(requestCode, data, callback);
if (!monriPaymentResult) {
    super.onActivityResult(requestCode, resultCode, data);
        }
    }
```

* result is delivered via `ResultCallback`
* collect Customer params and create `CustomerParams` instance
* create `TransactionParams` from `customerParams`, set other values if needed (eg override `order_info`)
* obtain `PaymentMethodsParams` from card via card.toPaymentMethodParams()
* create `ConfirmPaymentParams` from client\_secret, payment method params and transaction params
* invoke `monri.confirmPayment(context, confirmPaymentParams)`
* payment result will be returned to onActivityResult via setResult

_NOTE_ Values set in `TransactionParams` and `CustomerParams` will override those set in `payment/new` request.

CustomerParams:

| attribute  | length | type   | description                                                                        |
|------------|--------|--------|------------------------------------------------------------------------------------|
| full\_name | 3-30   | String | buyer's full name                                                                  |
| address    | 3-100  | String | buyer's address                                                                    |
| city       | 3-30   | String | buyer's city                                                                       |
| zip        | 3-9    | String | buyer's zip                                                                        |
| country    | 2      | String | buyer's country - [ISO two letter code](https://en.wikipedia.org/wiki/ISO\_3166-1) |
| phone      | 3-30   | String | buyer's phone                                                                      |
| email      | 3-100  | String | buyer's email                                                                      |

TransactionParams is used to override values set in `payment/new`.

***

Integration example is available in SDK's example and
on [this link](https://github.com/MonriPayments/monri-android/blob/master/app/src/main/java/com/monri/android/example/PaymentPickerActivity.java)

### Getting payment result on merchant's backend

Although you can easily collect payment result directly in application via `onSuccess` call it's better to implement
callback listener (WebHook) on your backend.

Requirements:

* it must be available over the Internet
* it must be secured (HTTPS)
* it must be set in merchant's setup on Monri's dashboard (if you are not able to set this value contact
  support@monri.com)

How it works:

* upon transaction processing and approval we'll send POST request to callback endpoint defined in merchant's settings
* validate received request to check if it's from us
* update/deliver order

Example of POST request sent to callback endpoint:

Body:

```json
{
  "id": 214,
  "acquirer": "integration_acq",
  "order_number": "3159daf002e3809",
  "amount": 100,
  "currency": "EUR",
  "ch_full_name": "John Doe",
  "outgoing_amount": 100,
  "outgoing_currency": "EUR",
  "approval_code": "687042",
  "response_code": "0000",
  "response_message": "approved",
  "reference_number": "000003036888",
  "systan": "000214",
  "eci": "06",
  "xid": null,
  "acsv": null,
  "cc_type": "visa",
  "status": "approved",
  "created_at": "2020-03-26T11:09:17.959+01:00",
  "transaction_type": "purchase",
  "enrollment": "N",
  "authentication": null,
  "pan_token": null,
  "masked_pan": "411111-xxx-xxx-1111",
  "issuer": "xml-sim",
  "number_of_installments": null,
  "custom_params": "{a:b, c:d}"
}
```

Headers:

| header              | value                                                                                                                                         |
|---------------------|-----------------------------------------------------------------------------------------------------------------------------------------------|
| accept-encoding     | gzip;q=1.0,deflate;q=0.6,identity;q=0.3                                                                                                       |
| authorization       | WP3-callback d5e4528ad8a0e0f4262e518c663d5ff83cd4a8f381db68f9d30f99961409ceebb719c16d423757fc36c532b902c987012f5825dc8d32dde3a9b7ed95876be77a |
| content-type        | application/json                                                                                                                              |
| http\_authorization | WP3-callback d5e4528ad8a0e0f4262e518c663d5ff83cd4a8f381db68f9d30f99961409ceebb719c16d423757fc36c532b902c987012f5825dc8d32dde3a9b7ed95876be77a |
| user-agent          | Faraday v0.15.4                                                                                                                               |
| content-length      | 621                                                                                                                                           |
| connection          | keep-alive                                                                                                                                    |

Where `authorization` and `http_authorization` headers are created as:

`digest = sha512(merchant_key + body)`

`authorization_header_value = WP3-callback digest`

To check if request is valid check:

* if authorization header schema is `WP3-callback`
* extract digest as second part
* verify digest
