---
cover: ../../.gitbook/assets/Blue Modern Marketing Manager LinkedIn Banner.png
coverY: 0
---

# 🟣 iOS SDK

The Monri iOS SDK makes it easy to build an excellent payment experience in your iOS app. It provides powerful, customizable, UI elements to use out-of-the-box to collect your users' payment details.

We also expose the low-level APIs that power those elements to make it easy to build fully custom forms. This guide will take you all the way from integrating our SDK to accepting payments from your users via credit cards.

### [Installation/Configuration](installation-guide.md) <a href="#user-content-installationconfiguration" id="user-content-installationconfiguration"></a>

#### Install and configure the SDK <a href="#user-content-install-and-configure-the-sdk" id="user-content-install-and-configure-the-sdk"></a>

You can choose to install the Monri iOS SDK via CocoaPods.

CocoaPods:

1. If you haven't already, install the latest version of [CocoaPods](https://guides.cocoapods.org/using/getting-started.html)
2. Add this line to your Podfile:

```
pod 'Monri', '~> 1.0'
```

3. Run the following command:

```
pod install
```

4. Don't forget to use the `.xcworkspace` file to open your project in Xcode, instead of the `.xcodeproj`file, from here on out.
5. In the future, to update to the latest version of the SDK, just run:

```
pod update Monri
```

For full installation/configuration guide check our wiki page at [Installation Guide](installation-guide.md)

## [Payment API Integration](payment-api-integration.md) <a href="#user-content-payment-api-integration" id="user-content-payment-api-integration"></a>

At some point in the flow of your app you'll obtain payment details from the user. After that you could:

* use obtained payment details and proceed with charge (confirmPayment)
* or tokenize obtained payment details for server side usage

In [Payment API Integration](payment-api-integration.md) it's explained how to:

* create payment
* collect payment details
* confirm payment
* get results back on your app and on your backend

If you want to tokenize obtained payment details then continue to the "Tokens API Integration"

## [Tokens API Integration](tokens-api-integration.md) <a href="#user-content-tokens-api-integration" id="user-content-tokens-api-integration"></a>

After you've obtained payment details its easy to securely transfer collected data via Tokens API.

In [Tokens API Integration](tokens-api-integration.md)) it's explained how to:

* create token request
* create token
* how to use created token for transaction authorization on your backend.
