# Tokens API Integration

Our iOS libraries let you easily accept mobile payments inside any iOS app.

Monri has created a Swift library for iOS, allowing you to easily submit payments from an iOS app.

Before you continue read [Installation Guide](https://github.com/MonriPayments/monri-ios/wiki/Installation-Guide) if you
have not already.

With our mobile library, we address PCI compliance by eliminating the need to send card data directly to your server.

Instead, our libraries send the card data directly to our servers, where we can:

* confirm payment with card data
* or convert card data to tokens

To learn how to convert data to tokens continue with this guide.

### [Create TokenRequest on merchant's backend](https://github.com/MonriPayments/monri-ios/wiki/Tokens-API-Integration#create-tokenrequest-on-merchants-backend) <a href="#user-content-create-tokenrequest-on-merchants-backend" id="user-content-create-tokenrequest-on-merchants-backend"></a>

This step is usually executed before presenting form for collecting payment details.

TokenRequest:

| field         | length     | type           | description                                                           |
|---------------|------------|----------------|-----------------------------------------------------------------------|
| random\_token | predefined | String or UUID | Token value                                                           |
| timestamp     | predefined | String         | Timestamp should be valid ISO date string                             |
| digest        | predefined | String         | Calculated as digest=`SHA512(merchant.key + random-token + timestamp` |

To create TokenRequest on our backend you'll need:

* `merchant_key` (available on merchant's dashboard)

Steps:

* create random\_token (UUID is sufficient)
* create timestamp
* create digest

Provide TokenRequest to the application to proceed with `monri.createToken` functionality

### [Converting card data to token on your app](https://github.com/MonriPayments/monri-ios/wiki/Tokens-API-Integration#converting-card-data-to-token-on-your-app) <a href="#user-content-converting-card-data-to-token-on-your-app" id="user-content-converting-card-data-to-token-on-your-app"></a>

After you've created TokenRequest on your backend simply proceed with:

* creating Monri instance
* collecting card details
* invoking monri.createToken for created TokenRequest

#### [Creating Monri instance](https://github.com/MonriPayments/monri-ios/wiki/Tokens-API-Integration#creating-monri-instance) <a href="#user-content-creating-monri-instance" id="user-content-creating-monri-instance"></a>

To create Monri instance you'll need:

* `authenticity_token` (available on merchant's dashboard)

```swift
// 1. initialize Monri
// Self is UIViewController
let monri = MonriApi(self, options: MonriApiOptions(authenticityToken: authenticityToken, developmentMode: true))

// 2. collect card data
@IBOutlet weak var cardInlineView: CardInlineView!

let card = cardInlineView.getCard() // or create Card instance yourself - Card(number: "4111 1111 1111 1111", cvc: "123", expMonth: 10, expYear: 2027)

// Optionally save card for future payments
card.tokenizePan = saveCardForFuturePaymentsSwitch.isOn

// 3. Validate card
if !card.validateCard() {
    print("Card validation failed")
    print("card.number valid = \(card.validateNumber())")
    print("card.cvv valid = \(card.validateCVC())")
    print("card.exp_date valid = \(card.validateExpiryDate())")
    // Card validation failed
}

// 4. Invoke createToken

monri.createToken(tokenRequest, paymentMethod: card) {
    result in
    switch result {
    case .error(let error):
        print("An error occurred \(error)")
    case .token(let token):
        print("Token received \(token)")
    }
}
```

### [Submitting Token to Merchant's backend](https://github.com/MonriPayments/monri-ios/wiki/Tokens-API-Integration#submitting-token-to-merchants-backend) <a href="#user-content-submitting-token-to-merchants-backend" id="user-content-submitting-token-to-merchants-backend"></a>

After invoking `createToken` obtained card details are now represented by token.

To use token for transaction authorization simply submit token to your backend.

### [Using Token for Transaction Authorization](https://github.com/MonriPayments/monri-ios/wiki/Tokens-API-Integration#using-token-for-transaction-authorization) <a href="#user-content-using-token-for-transaction-authorization" id="user-content-using-token-for-transaction-authorization"></a>

After you have:

* created TokenRequest
* obtained card data and converted to token via `monri.createToken`
* submitted token to your backend

You can use token to execute transaction authorization.

Continue with this guide to learn how to use token for transaction authorization.

### [Requirements](https://github.com/MonriPayments/monri-ios/wiki/Tokens-API-Integration#requirements) <a href="#user-content-requirements" id="user-content-requirements"></a>

Integration must be done within our test environment first. When this process is finished and approved by our staff, you
may go live and start processing with real money.

To start integrating with WebPay service you will need:

* test merchant account
* HTTP client library

If you don't have a test merchant account, please contact us at [support@monri.com](mailto:support@monri.com) and we
will open one for you. Then you can login into your account
at [https://ipgtest.monri.com/ba-hr/login](https://ipgtest.monri.com/ba-hr/login) with login and password provided.

### [Transactions API](https://github.com/MonriPayments/monri-ios/wiki/Tokens-API-Integration#transactions-api) <a href="#user-content-transactions-api" id="user-content-transactions-api"></a>

#### [Variables - names, lengths and formats](https://github.com/MonriPayments/monri-ios/wiki/Tokens-API-Integration#variables---names-lengths-and-formats) <a href="#user-content-variables---names-lengths-and-formats" id="user-content-variables---names-lengths-and-formats"></a>

Here are the variables and their definitions used when generating JSON documents for API calls:

#### [Buyer's profile](https://github.com/MonriPayments/monri-ios/wiki/Tokens-API-Integration#buyers-profile) <a href="#user-content-buyers-profile" id="user-content-buyers-profile"></a>

| name           | length | format       | additional info   |
|----------------|--------|--------------|-------------------|
| ch\_full\_name | 3-30   | alphanumeric | buyer's full name |
| ch\_address    | 3-100  | alphanumeric | buyer's address   |
| ch\_city       | 3-30   | alphanumeric | buyer's city      |
| ch\_zip        | 3-9    | alphanumeric | buyer's zip       |
| ch\_country    | 3-30   | alphanumeric | buyer's country   |
| ch\_phone      | 3-30   | alphanumeric | buyer's phone     |
| ch\_email      | 3-100  | alphanumeric | buyer's email     |

#### [Card details](https://github.com/MonriPayments/monri-ios/wiki/Tokens-API-Integration#card-details) <a href="#user-content-card-details" id="user-content-card-details"></a>

| name           | length | format       | additional info                                                                                       |
|----------------|--------|--------------|-------------------------------------------------------------------------------------------------------|
| temp\_card\_id | 0-40   | alphanumeric | value representing tokenized card data - `THIS IS YOUR TOKEN (same one returned by createToken call)` |

#### [Order details](https://github.com/MonriPayments/monri-ios/wiki/Tokens-API-Integration#order-details) <a href="#user-content-order-details" id="user-content-order-details"></a>

| name          | length     | format       | additional info                                         |
|---------------|------------|--------------|---------------------------------------------------------|
| order\_info   | 3-100      | alphanumeric | short description of order being processed              |
| order\_number | 1-40       | alphanumeric | unique identifier                                       |
| amount        | 3-11       | integer      | amount is in minor units, ie. 10.24 USD is sent as 1024 |
| currency      | predefined | alpha        | possible values are USD, EUR, BAM or HRK                |

#### [Processing data](https://github.com/MonriPayments/monri-ios/wiki/Tokens-API-Integration#processing-data) <a href="#user-content-processing-data" id="user-content-processing-data"></a>

| name                     | length     | format       | additional info                                                                                                                          |
|--------------------------|------------|--------------|------------------------------------------------------------------------------------------------------------------------------------------|
| ip                       | 7-15       | alphanumeric | valid IPv4 address                                                                                                                       |
| language                 | predefined | alpha        | used for errors localization, possible values are en, es, ba or hr                                                                       |
| transaction\_type        | predefined | alpha        | possible values are authorize, purchase, capture, refund, void                                                                           |
| authenticity\_token      | 40         | alphanumeric | auto generated value for merchant account, can be found under merchant settings                                                          |
| digest                   | 40         | alphanumeric | SHA512 hash generated from concatenation of key, order\_number, amount and currency as strings; key can be found under merchant settings |
| number\_of\_installments | 1-2        | integer      | range 2-12                                                                                                                               |
| moto                     | predefined | boolean      | possible value is true or false; missing variable is equivalent to false                                                                 |

#### [Transaction messages](https://github.com/MonriPayments/monri-ios/wiki/Tokens-API-Integration#transaction-messages) <a href="#user-content-transaction-messages" id="user-content-transaction-messages"></a>

#### [Authorization](https://github.com/MonriPayments/monri-ios/wiki/Tokens-API-Integration#authorization) <a href="#user-content-authorization" id="user-content-authorization"></a>

Authorization is a preferred transaction type for e-commerce. Merchant must capture these transactions within 28 days in
order to transfer the money from buyer's account to his own. This transaction can also be voided if buyer cancel the
order. Refund can be done after original authorization is captured.

Below is an JSON example of authorization message in which transaction\_type tag has a value authorize. This JSON
document is generated according
to [variable definitions](https://ipgtest.monri.com/ba-hr/documentation/direct#variables).

Digest is calculated using following formula:

_digest = SHA512(key + order\_number + amount + currency)_

With the following example data

* key: qwert123
* order\_number: abcdef
* amount: 54321
* currency: EUR

the digest formula gives a result as follows:

`digest = SHA512("qwert123abcdef54321EUR") = "5cb109cd348b74824c29a46fc029b6b7d3fc2c34835b834f276451c8c48c5d921b9a85fa1701ed01d2031b81f998ecbd99707df6e9e0a1087f40f4f82aacf514"`

`key` is a shared secret used to calculate digest value. It is set through merchant interface under API settings of your
merchant `account.authenticity_token` is auto generated value and is copied from merchant account.

`NOTICE` Client does not send a TID/MID pair in authorization message, those are set in merchant account.

[**Authorization request example
**](https://github.com/MonriPayments/monri-ios/wiki/Tokens-API-Integration#authorization-request-example)

```json5
{
  "transaction": {
    "transaction_type": "authorize",
    "amount": 100,
    "ip": "10.1.10.111",
    "order_info": "Monri components trx",
    "ch_address": "Adresa",
    "ch_city": "Grad",
    "ch_country": "BIH",
    "ch_email": "test@test.com",
    "ch_full_name": "Test",
    "ch_phone": "061 000 000",
    "ch_zip": "71000",
    "currency": "BAM",
    "digest": "6af94189788cc073464764c69a9afaea3196bd7ca84b442a76aa141f6b48f8cf323b77aebe0cb72099816e69c08981eccd312ff30d88fa293670a830d78e2466",
    "order_number": "1568236677437",
    "authenticity_token": "6a13d79bde8da9320e88923cb3472fb638619ccb",
    "language": "en",
    "temp_card_id": "5a91058d-a9c0-46b3-aa02-ef1bb2255711"
  }
}
```

This JSON is now posted to `https://ipgtest.monri.com/v2/transaction`.

`IMPORTANT` Parametrize `https://ipgtest.monri.com` URL, in production mode the subdomain will be different.

If all values pass validations at our side, transaction is send to the bank and response is returned. This response may
look like this:

* **HTTP status code:** 201 - Created
* **HTTP headers:** {:connection=>"close", :date=>"Tue, 25 Oct 2011 01:18:37 GMT", :
  location=>"[https://ipgtest.monri.com/transactions/845](https://ipgtest.monri.com/transactions/845)", :
  content\_type=>"application/json; charset=utf-8", :cache\_control=>"no-cache", :x\_ua\_compatible=>"IE=Edge", :
  x\_runtime=>"1.475305", :transfer\_encoding=>"chunked"}
* **HTTP body:**

```json
{
  "transaction": {
    "id": 187091,
    "acquirer": "integration_acq",
    "order_number": "1568265143783",
    "amount": 100,
    "currency": "BAM",
    "outgoing_amount": 100,
    "outgoing_currency": "BAM",
    "approval_code": "376161",
    "response_code": "0000",
    "response_message": "approved",
    "reference_number": "000002903748",
    "systan": "187090",
    "eci": "06",
    "xid": null,
    "acsv": null,
    "cc_type": "visa",
    "status": "approved",
    "created_at": "2019-09-12T07:12:27.019+02:00",
    "transaction_type": "purchase",
    "enrollment": "N",
    "authentication": null,
    "pan_token": null,
    "issuer": "xml-sim"
  }
}
```

New transaction is generated - _201 Created HTTP status code_, and it's location is set in appropriate HTTP header. A
client then must parse a body from HTTP response and extract all values from that `JSON` document. Transaction is
approved only and if only status is set to `approved`. All other fields are standard data carried over payment networks.
If issuer declines a transaction, status flag is set to `declined`. In a case of an error, the flag will be set
to `invalid`.

`IMPORTANT` Do not rely on any output variable except status to determine successful of authorization.

IMPORTANT authorize messages won't be settled unless they
are [captured](https://ipgtest.monri.com/ba-hr/documentation/direct#capture) within 28 days. After authorization is
captured, it can be [refunded](https://ipgtest.monri.com/ba-hr/documentation/direct#refund) within 180 days.

`NOTICE` We highly recommend to our merchants to keep a whole response (this includes HTTP headers and body) and to save
all parsed values for easier troubleshooting during the integration phase and production later on. Even if the body is
empty, HTTP response code is valuable information; HTTP headers are in the hearth of REST architecture. The quality of
our support depends on availability of these information.

In case of invalid request, service will also return a response with _422 Unprocessable Entity HTTP status code_ and
JSON document in its body. Each offended variable will be printed out along with brief explanation what went wrong. That
response may look like this:

```http
{
   "errors":[
      "Ch email is invalid"
   ]
}
```

This invalid request is also recorded and errors are visible through merchant account interface.

#### [Purchase](https://github.com/MonriPayments/monri-ios/wiki/Tokens-API-Integration#purchase) <a href="#user-content-purchase" id="user-content-purchase"></a>

Purchase doesn't need to be approved, funds are transfered in next settlement between issuer and acquirer banks, usually
within one business day. These transactions can be refunded within 180 days.

This message has the same structure as
authorization [request](https://ipgtest.monri.com/en/documentation/direct#authorization-request) JSON document, only
difference is in transaction\_type tag which has _purchase_ value now. Response has identical structure as
authorization [response](https://ipgtest.monri.com/en/documentation/direct#authorization-response) and all response
fields should be treated in the same way.

NOTICE purchase message can be [refunded](https://ipgtest.monri.com/en/documentation/direct#refund) within 180 days.

#### [Capture](https://github.com/MonriPayments/monri-ios/wiki/Tokens-API-Integration#capture) <a href="#user-content-capture" id="user-content-capture"></a>

Refer to [direct api documentation](https://ipgtest.monri.com/en/documentation/direct#capture).

#### [Refund](https://github.com/MonriPayments/monri-ios/wiki/Tokens-API-Integration#refund) <a href="#user-content-refund" id="user-content-refund"></a>

Refer to [direct api documentation](https://ipgtest.monri.com/en/documentation/direct#refund)

#### [Void](https://github.com/MonriPayments/monri-ios/wiki/Tokens-API-Integration#void) <a href="#user-content-void" id="user-content-void"></a>

Refer to [direct api documentation](https://ipgtest.monri.com/en/documentation/direct#void)

#### [3-D Secure messages](https://github.com/MonriPayments/monri-ios/wiki/Tokens-API-Integration#3-d-secure-messages) <a href="#user-content-3-d-secure-messages" id="user-content-3-d-secure-messages"></a>

If your merchant account has active 3-DS flag under its settings, all incoming authorize and purchase requests will be
processed as 3-D Secure transactions. For cards not enrolled in 3-DS, a regular authorize or
purchase [response](https://ipgtest.monri.com/ba-hr/documentation/direct#authorization-response) will be returned.

If the card is enrolled, a 3-DS check will occur and appropriate response is returned which may look like this:

* **HTTP status code:** 201 - Created
* **HTTP headers:** {:connection=>"close", :date=>"Wed, 26 Oct 2011 15:39:18 GMT", :content\_type=>"application/json;
  charset=utf-8", :cache\_control=>"no-cache", :x\_ua\_compatible=>"IE=Edge", :x\_runtime=>"4.147298", :
  transfer\_encoding=>"chunked"}
* **HTTP body:**

```http
{
   "secure_message":{
      "id":1870801,
      "acs_url":"https://ipgtest.monri.com/three_ds_roundtrip/4efa608821e763f488a14f152b7e762d/collect_browser_info",
      "pareq":"PareqHandlingByClientIsDiscontinued",
      "authenticity_token":"4efa608821e763f488a14f152b7e762d"
   }
}
```

Client should parse a HTTP body from above example response and extracts _acs\_url_, _pareq_ and _authenticity\_token_
values. They are POST-ed through buyer's browser to ACS server at _acs-url_ as follows (this example use javascript to
automatically submit the form):

```html
 <!DOCTYPE html>
<html>

<head>
    <title>3D Secure Verification</title>
    <script language="Javascript">
        function OnLoadEvent() {
            document.form.submit();
        }
    </script>
</head>

<body OnLoad="OnLoadEvent();">
Invoking 3-D secure form, please wait ...
<form name="form" action="_acs-url_" method="post">
    <input type="hidden" name="PaReq" value="_pareq_">
    <input type="hidden" name="TermUrl" value="_term-url_">
    <input type="hidden" name="MD" value="_authenticity-token_">
    <noscript>
        <p>Please click</p>
        <input id="to-acs-button" type="submit">
    </noscript>
</form>
</body>

</html>
```

where `_acs-url_`, `_pareq_` and `_authenticity-token_` are substituted with appropriate extracted values.

Buyer will POST the result of 3-D secure identity check from ACS server to _term-url_ at merchant side through his
browser. Following data is captured at merchant's term-url:

* PaRes - eJzNmGuvosyygP/KZPZHMy93lYmzkuaOCnJH+LLDTe6ggoD8+t3qrDXrnUxO3rO/nENigKK6urq76qm2N1Z2TR ...
* MD - 7465c9ab97defa1501ed0e680b3a0b4b88937c17

`NOTICE` Merchant should implement a listener at _term-url_ that captures response from issuer's ACS server.

The 3-D secure processing is done and merchant now issue a new request to [
_https://ipgtest.monri.com/pares_](https://ipgtest.monri.com/pares) to finish the transaction. Example of such request
may look like this:

```
{
   "PaRes":"ParesHandlingByClientIsDiscontinued",
   "MD":"d73144603639e19c1cb0f247000860f1"
}
```

`NOTICE` authenticity-token from WebPay is submitted to ACS server in variable MD; then is sent back again to WebPay as
MD variable.

WebPay will return [response](https://ipgtest.monri.com/ba-hr/documentation/direct#authorization-response) as would for
a regular authorize or purchase request messages. Only difference is that _eci_, _xid_, _acsv_, _enrollment_ and
_authentication_ fields are now populated in response JSON according to 3-DS rules.

#### [List of response codes](https://github.com/MonriPayments/monri-ios/wiki/Tokens-API-Integration#list-of-response-codes) <a href="#user-content-list-of-response-codes" id="user-content-list-of-response-codes"></a>

Here is the list of response codes and their description:

* 0000 - Approved
* 1001 - Card expired
* 1002 - Card suspicious
* 1003 - Card suspended
* 1004 - Card stolen
* 1005 - Card lost
* 1011 - Card not found
* 1012 - Cardholder not found
* 1014 - Account not found
* 1015 - Invalid request
* 1016 - Not sufficient funds
* 1017 - Previously reversed
* 1018 - Previously reversed
* 1019 - Further activity prevents reversal
* 1020 - Further activity prevents void
* 1021 - Original transaction has been voided
* 1022 - Preauthorization is not allowed for this card
* 1023 - Only full 3D authentication is allowed for this card
* 1024 - Installments are not allowed for this card
* 1025 - Transaction with installments can not be send as preauthorization
* 1026 - Installments are not allowed for non ZABA cards
* 1050 - Transaction declined
* 1802 - Missing fields
* 1803 - Extra fields exist
* 1804 - Invalid card number
* 1806 - Card not active
* 1808 - Card not configured
* 1810 - Invalid amount
* 1811 - System error - database
* 1812 - System error - transaction
* 1813 - Cardholder not active
* 1814 - Cardholder not configured
* 1815 - Cardholder expired
* 1816 - Original not found
* 1817 - Usage limit reached
* 1818 - Configuration error
* 1819 - Invalid terminal
* 1820 - Inactive terminal
* 1821 - Invalid merchant
* 1822 - Duplicate entity
* 1823 - Invalid acquirer
* 2000 - Internal error - host down
* 2001 - Internal error - host timeout
* 2002 - Internal error - invalid message
* 2003 - Internal error - message format error
* 2013 - 3D Secure error - invalid request
* 3000 - Time expired
* 3100 - Function not supported
* 3200 - Timeout
* 3201 - Authorization host not active
* 3202 - System not ready
* 4001 - 3D Secure error - ECI 7
* 4002 - 3D Secure error - not 3D Secure, store policy
* 4003 - 3D secure error - not authenticated
* 5000 - Request in progress
* 5018 - RISK: Minimum amount per transaction
* 5019 - RISK: Maximum amount per transaction
* 5001 - RISK: Number of repeats per PAN
* 5020 - RISK: Number of approved transactions per PAN
* 5003 - RISK: Number of repeats per BIN
* 5016 - RISK: Total sum on amount
* 5021 - RISK: Sum on amount of approved transactions per PAN
* 5022 - RISK: Sum on amount of approved transactions per BIN
* 5005 - RISK: Percentage of declined transactions
* 5009 - RISK: Number of chargebacks
* 5010 - RISK: Sum on amount of chargebacks
* 5006 - RISK: Number of refunded transactions
* 5007 - RISK: Percentage increment of sum on amount of refunded transactions
* 5023 - RISK: Number of approved transactions per PAN and MCC on amount
* 5011 - RISK: Number of retrieval requests
* 5012 - RISK: Sum on amount of retrieval requests
* 5013 - RISK: Average amount per transaction
* 5014 - RISK: Percentage increment of average amount per transaction
* 5015 - RISK: Percentage increment of number of transactions
* 5017 - RISK: Percentage increment of total sum on amount
* 5050 - RISK: Number of repeats per IP
* 5051 - RISK: Number of repeats per cardholder name
* 5052 - RISK: Number of repeats per cardholder e-mail
* 6000 - Systan mismatch

### [Next steps](https://github.com/MonriPayments/monri-ios/wiki/Tokens-API-Integration#next-steps)&#x20;

Congrats! You now have a custom payment form to accept card payments with Monri. Once you’ve sent your form to your
server, you’ll be able to use the token to perform a charge or to save to a customer.

### [Authorize example](https://github.com/MonriPayments/monri-ios/wiki/Tokens-API-Integration#authorize-example) <a href="#user-content-authorize-example" id="user-content-authorize-example"></a>

Example of action handling form submit:

```php
function transactionData($transactionType) {
    $amount = '100';
    $sec = new Security();
    $currency = 'EUR';
    $order_number = "monri-components" . $sec->generateRandomString(10);
    //digest = SHA512(key + order_number + amount + currency)
    $digest = hash('sha512', $key . $order_number . $amount . $currency);
    return  [
        "transaction_type" => $transactionType,
        "amount" => $amount,
        "ip" => '10.1.10.111',
        'order_info' => 'Monri components trx',
        'ch_address' => 'Adresa',
        'ch_city' => 'Grad',
        'ch_country' => 'BIH',
        'ch_email' => 'test@test.com',
        'ch_full_name' => 'Test',
        'ch_phone' => '061 000 000',
        'ch_zip' => '71000',
        'currency' => $currency,
        'digest' => $digest,
        'order_number' => $order_number,
        'authenticity_token' => $authenticity_token,
        'language' => 'en',
        // This part is important! Extract monriToken from post body
        'temp_card_id' => Yii::$app->request->post('monriToken')
     ];
}

function transaction($url, $data) {
    $data_string = Json::encode(['transaction' => $data]);
// Execute transaction
    $ch = curl_init($url . './v2/transaction');
      curl_setopt($ch, CURLOPT_CUSTOMREQUEST, "POST");
      curl_setopt($ch, CURLOPT_POSTFIELDS, $data_string);
      curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);
      curl_setopt($ch, CURLOPT_HTTPHEADER, array(
      'Content-Type: application/json',
      'Content-Length: ' . strlen($data_string))
     );
    // TODO: handle transaction result
    $result = curl_exec($ch);
    return $result;
}

function transactionExample() {
    $url = 'https://ipgtest.monri.com'; // change for production env
    // Prepare transaction payload, include monriToken as `temp-card-id` field
    $data = transactionData('authorize');
    return transaction($url, $data);
}
```

### [Purchase example](https://github.com/MonriPayments/monri-ios/wiki/Tokens-API-Integration#purchase-example) <a href="#user-content-purchase-example" id="user-content-purchase-example"></a>

```php
function transactionExample() {
    $url = 'https://ipgtest.monri.com'; // change for production env
    // Prepare transaction payload, include monriToken as `temp-card-id` field
    $data = transactionData('purchase');
    return transaction($url, $data);
}
```
