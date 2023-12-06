---
cover: ../../.gitbook/assets/cover.png
coverY: 0
layout:
  cover:
    visible: true
    size: full
  title:
    visible: true
  description:
    visible: true
  tableOfContents:
    visible: true
  outline:
    visible: true
  pagination:
    visible: true
---

# Tokens API Integration

Our Flutter libraries let you easily accept mobile payments and manage customer information inside any Android app.

Monri has created a Java library for Android, allowing you to easily submit payments from an Android app. With our
mobile library, we address PCI compliance by eliminating the need to send card data directly to your server. Instead,
our libraries send the card data directly to our servers, where we can convert them to tokens.

Your app will receive the token back, and can then send the token to an endpoint on your server, where it can be used to
process a payment.

### Collecting credit card information

At some point in the flow of your app, you’ll want to obtain payment details from the user. There are a couple ways to
do this:

* Use our built-in card input widget to collect card information
* Build your own credit card form

Instructions for each route follows, although you may want to write your app to offer support for both.

### Using the card input widget

To collect card data from your customers directly, you can use

This allows your customers to input all the required data for their card: the number, the expiration date, and the CVV
code. Note that the value of the `Card` object is `null` if the data in the widget is either incomplete or fails
client-side validity checks.

```java
new TextFormField(
  keyboardType: TextInputType.number,
  focusNode: _cardNumberFocusNode,
  onFieldSubmitted: (_) {
    FocusScope.of(context).requestFocus(_cvvFocusNode);
  },
  inputFormatters: [
    FilteringTextInputFormatter.digitsOnly,
    new LengthLimitingTextInputFormatter(19),
    new CardNumberInputFormatter()
  ],
  controller: numberController,
  decoration: new InputDecoration(
    enabledBorder: OutlineInputBorder(
      borderRadius: BorderRadius.circular(10.0),
      borderSide:  BorderSide(color: Colors.transparent),
    ),
    filled: true,
    icon: CardUtils.getCardIcon(_cardType),
    labelText: 'Card Number',
  ),
  style: TextStyle(
      color: widget.objectArgument.threeDS ? Colors.black45 : Colors.blue
  ),
  enabled: widget.objectArgument.threeDS? false : true,
  onSaved: (String? value) {
    _cardNumber = CardUtils.getCleanedNumber(value!);
  },
  validator: CardUtils.validateCardNum,
),
```

### Building your own form

If you build your own payment form, you’ll need to collect at least your customers’ card numbers and expiration dates.
Monri strongly recommends collecting the CVC. You can optionally collect the user’s name and billing address for
additional fraud protection.

Once you’ve collected a customer’s information, you will need to exchange the information for a Monri token.

#### Creating & validating cards from a custom form

To create a `Card` object from data you’ve collected from other forms, you can create the object with its constructor.

```java
import 'package:MonriPayments/src/payment_method.dart';

class Card extends PaymentMethod {
  String number;
  String cvc;
  int expMonth;
  int expYear;
  bool tokenizePan;

  Card(this.number, this.cvc, this.expMonth, this.expYear, this.tokenizePan);

  @override
  String paymentMethodType() => PaymentMethod.TYPE_CARD;

  @override
  Map<String, String> data() {
    var data = Map<String, String>();
    data["pan"] = number;
    data["expiration_date"] = "$expYear$expMonth";
    data["cvv"] = cvc;
    data["tokenize_pan"] = "$tokenizePan";
    return data;
  }
}
```

As you can see in the example above, the `Card` instance contains some helpers to validate that the card number passes
the Luhn check, that the expiration date is the future, and that the CVC looks valid. You’ll probably want to validate
these three things at once, so we’ve included a `validateCard` function that does so.

```java
  static String? validateCardNum(String? input) {
    if (input == null || input.isEmpty) {
      return ValidationMessages.filedRequired;
    }

    input = getCleanedNumber(input);

    if (input.length < 8) {
      return ValidationMessages.invalidCardNumber;
    }

    int sum = 0;
    int length = input.length;
    for (var i = 0; i < length; i++) {
      // get digits in reverse order
      int digit = int.parse(input[length - i - 1]);

      // every 2nd number multiply with 2
      if (i % 2 == 1) {
        digit *= 2;
      }
      sum += digit > 9 ? (digit - 9) : digit;
    }

    if (sum % 10 == 0) {
      return null;
    }

    return ValidationMessages.invalidCardNumber;
  }
```

#### Tokens API

```java
final Monri monri = new Monri(
    getContext(),
    "authenticity_token",
);

final TokenRequest tokenRequest = new TokenRequest(
  "random-token", // Random UUID
  "digest", // SHA512{merchant.key}\#{random-token}\#{timestamp}()
  "timestamp" // valid ISO date string
);

monri.createToken(
	tokenRequest,
    	card,
    	new TokenCallback() {
        	public void onSuccess(@NonNull Token token) {
	        // Send token to your server
        	}

	        public void onError(@NonNull Exception error) {
	        // Show localized error message
            	Toast.makeText(
			getContext(),
                	error.getLocalizedString(),
                	Toast.LENGTH_LONG
            	).show();
        }
    }
)
```

> Authenticity token should be replaced with live authenticity token in production.

### Using tokens

Using the payment token, however it was obtained, requires an API call from your server using your secret merchant
key. (For security purposes, you should never embed your secret merchant key in your app.)

Set up an endpoint on your server that can receive an HTTP POST call for the token. In the `onSuccess` callback (when
using your own form), you’ll need to POST the supplied token to your server. Make sure any communication with your
server is [SSL secured](https://ipgtest.monri.com/en/security) to prevent eavesdropping.
