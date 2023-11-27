---
cover: ../../.gitbook/assets/Blue Modern Marketing Manager LinkedIn Banner.png
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

# 🟣 Flutter SDK

The Monri Flutter SDK makes it easy to build an excellent payment experience in your Android/iOS app. It provides powerful, customizable, UI elements to use out-of-the-box to collect your users' payment details.

We also expose the low-level APIs that power those elements to make it easy to build fully custom forms. This guide will take you all the way from integrating our SDK to accepting payments from your users via credit cards.

## Payment API Integration

At some point in the flow of your app you'll obtain payment details from the user. After that you could:

* use obtained payment details and proceed with charge (confirmPayment)
* or tokenize obtained payment details for server side usage

In [Payment API Integration](payment-api-integration.md) it's explained how to:

* create payment
* collect payment details
* confirm payment
* get results back on your app and on your backend

If you want to tokenize obtained payment details then continue to the "Tokens API Integration"

## Tokens API Integration

After you've obtained payment details it's easy to securely transfer collected data via Tokens API.

In [Tokens API Integration](tokens-api-integration.md) it's explained how to:

* create token request
* create token
* how to use created token for transaction authorization on your backend

## Installation Guide

In pubspec.yaml add

```yaml
dependencies:
  MonriPayments:
    git:
      url: https://github.com/MonriPayments/flutter-monri-android-ios.git
```

**Android Gradle**

On your `build.gradle` file add this statement to the `dependencies` section:

```groovy
buildscript {
    //...
    dependencies {
        classpath 'com.android.tools.build:gradle:7.0.2'
    }
}
```

### Usage

```dart
import 'package:MonriPayments/MonriPayments.dart';
```

```dart
// ...
final monriPayments = MonriPayments.create();

Future<void> _continuePayment() async {
  Map data = {};
  // Platform messages may fail, so we use a try/catch PlatformException.
  try {
    var clientSecret = "client_secret"; // create one on your backend
    var arguments = jsonDecode(_getJsonData(
        isDevelopment: true,
        clientSecret: clientSecret,
        cardNumber: "4341792000000044",
        cvv: 123,
        expirationMonth: 12,
        expirationYear: 2030,
        cardHolderName: "Harun Kolos",
        tokenizePan: true
    ));
    data = (await monriPayments.confirmPayment(CardConfirmPaymentParams.fromJSON(arguments))).toJson();
  } on PlatformException {
    data = {};
  }

}

String _getJsonData({
  required String clientSecret,
  required bool isDevelopment,
  required String cardNumber,
  required int cvv,
  required int expirationMonth,
  required int expirationYear,
  required String cardHolderName,
  required bool tokenizePan
}){
  return """
    {
        "is_development_mode": $isDevelopment,
        "authenticity_token": "a6d41095984fc60fe81cd3d65ecafe56d4060ca9", //available on merchant's dashboard
        "client_secret": "$clientSecret",
        "card": {
        "pan": "$cardNumber",
            "cvv": "$cvv",
            "expiryMonth": "$expirationMonth",
            "expiryYear": "$expirationYear",
            "tokenize_pan": $tokenizePan
    },
        "transaction_params": {
        "full_name": "$cardHolderName",
            "address": "N/A",
            "city": "Sarajevo",
            "zip": "71000",
            "phone": "N/A",
            "country": "BA",
            "email": "flutter@monri.com",
            "custom_params": ""
    }
    }
    """;
}
```
