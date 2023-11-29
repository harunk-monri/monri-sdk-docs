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

# 🟣 Ionic SDK

The Monri Ionic SDK makes it easy to build an excellent payment experience in your Android/iOS app. It provides
powerful, customizable, UI elements to use out-of-the-box to collect your users' payment details.

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

Clone project in the same tree view as your project. In `package.json` add

In `package.json` add

```json
{
  "dependencies": {
    "ionic-monri-android-ios": "file:../ionic-monri-android-ios"
  }
}
```

```shell
npm install
npx cap sync
```

### Usage

```javascript
import {IonicMonri} from '../../../../ionic-monri-android-ios';

// ...
//new card
const result = await IonicMonri.confirmPayment({
        options: {
            authenticityToken: '6a13d79bde8da9320e88923cb3472fb638619ccb',
            developmentMode: true,
        },
        params: {
            clientSecret: "client_secret", // create one on your backend
            card: {
                pan: '4111 1111 1111 1111',
                cvv: '123',
                expiryMonth: 12,
                expiryYear: 2032,
                saveCard: true
            },
            transaction: {
                email: 'ionic.monri@gmail.com',
                orderInfo: 'Ionic monri order info',
                phone: '061123213',
                city: 'Sarajevo',
                country: 'BA',
                address: 'Ferhadija',
                fullName: 'Ionic Monri Example',
                zip: '71210',
            },
        }
    }
);

//saved card
const result = await IonicMonri.confirmPayment({
        options: {
            authenticityToken: '6a13d79bde8da9320e88923cb3472fb638619ccb',
            developmentMode: true,
        },
        params: {
            clientSecret: "client_secret", // create one on your backend
            savedCard: {
                panToken: 'd5719409d1b8eb92adae0feccd2964b805f93ae3936fdd9d8fc01a800d094584', //retrive one via API
                cvv: '123',//allow the user to enter cvv..
            },
            transaction: {
                email: 'ionic.saved.card.monri@gmail.com',
                orderInfo: 'Ionic monri order info saved card no 3DS',
                phone: '061123213',
                city: 'Sarajevo',
                country: 'BA',
                address: 'Ferhadija',
                fullName: 'Ionic Monri Example',
                zip: '71210',
            },
        }
    }
);
```
