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

# 🟣 ReactNative SDK

The Monri React-Native SDK makes it easy to build an excellent payment experience in your Android/iOS app. It provides powerful, customizable, UI elements to use out-of-the-box to collect your users' payment details.

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

After you've obtained payment details its easy to securely transfer collected data via Tokens API.

In [Tokens API Integration](tokens-api-integration.md) it's explained how to:

* create token request
* create token
* how to use created token for transaction authorization on your backend

## Installation Guide

```shell
npm install react-native-monri-android-ios
```

### Usage

```javascript
import MonriAndroidIos from "react-native-monri-android-ios";

// ...

const result = await MonriAndroidIos.confirmPayment({
    authenticityToken: '6a13d79bde8da9320e88923cb3472fb638619ccb',
    developmentMode: true,
  },
  {
    clientSecret: "client_secret", // create one on your backend
    card: {
      pan: '4341 7920 0000 0044',
      cvv: '123',
      expiryMonth: 12,
      expiryYear: 2032,
      saveCard: true
    },
    transaction: {
      email: 'test-react-native@monri.com',
      orderInfo: 'React native bridge???',
      phone: '061123213',
      city: 'Sarajevo',
      country: 'BA',
      address: 'Radnicka',
      fullName: 'Test Test',
      zip: '71210',
    },
  });
```
