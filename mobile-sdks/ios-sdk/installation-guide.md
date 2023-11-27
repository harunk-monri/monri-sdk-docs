# Installation Guide

The Monri iOS SDK makes it easy to build an excellent payment experience in your iOS app. It provides powerful,
customizable, UI elements to use out-of-the-box to collect your users' payment details.

We also expose the low-level APIs that power those elements to make it easy to build fully custom forms. This guide will
take you all the way from integrating our SDK to accepting payments from your users via credit cards.

### [Install and configure the SDK](https://github.com/MonriPayments/monri-ios/wiki/Installation-Guide#install-and-configure-the-sdk) <a href="#user-content-install-and-configure-the-sdk" id="user-content-install-and-configure-the-sdk"></a>

You can choose to install the Monri iOS SDK via CocoaPods.

CocoaPods:

1. If you haven't already, install the latest version
   of [CocoaPods](https://guides.cocoapods.org/using/getting-started.html)
2. Add this line to your Podfile:

```
pod 'Monri', '~> 1.0'
```

3. Run the following command:

```
pod install
```

4. Don't forget to use the `.xcworkspace` file to open your project in Xcode, instead of the `.xcodeproj`file, from here
   on out.
5. In the future, to update to the latest version of the SDK, just run:

```
pod update Monri
```

## [Collecting credit card information](https://github.com/MonriPayments/monri-ios/wiki/Installation-Guide#collecting-credit-card-information) <a href="#user-content-collecting-credit-card-information" id="user-content-collecting-credit-card-information"></a>

At some point in the flow of your app, you’ll want to obtain payment details from the user. There are a couple ways to
do this:

* [Use our built-in card input to collect card information](https://monri.com/docs/mobile/ios#card-input)
* [Build your own credit card form](https://monri.com/docs/mobile/ios#credit-card-form)

Instructions for each route follows, although you may want to write your app to offer support for both.

### [Using the card input](https://github.com/MonriPayments/monri-ios/wiki/Installation-Guide#using-the-card-input) <a href="#user-content-using-the-card-input" id="user-content-using-the-card-input"></a>

To collect card data from your customers directly, you can use
Monri’s [CardInlineView](https://github.com/jasminsuljic/monri-ios/blob/master/Monri/Classes/CardInlineView.swift). Yo
can include it in any view:

* Add a UITextField to your view in interface builder and change its class to `CardInlineView` (when using interface
  builder)
* or initate a CardInlineView with one of its initializers (when instantiating from code):
    * init?(coder: aDecoder: NSCoder)
    * init(frame: CGRect) To get updates about entered card information on your view controller, add card to your view
      controller

```swift
class ViewController: UIViewController {
  @IBOutlet weak var cardInlineView: CardInlineView!
}
```

This allows your customers to input all of the required data for their card: the number, the expiration date, and the
CVV code.

```swift
var card = cardInlineView.getCard()
if !card.validateCard() {
  // card invalid, show appropriate validation error
  // If needed fields can be validated separately
  print("Card validation failed")
  print("card.number valid = \(card.validateNumber())")
  print("card.cvv valid = \(card.validateCVC())")
  print("card.exp_date valid = \(card.validateExpiryDate())")
  // Card validation failed
} else {
  // continue with token creation
}
```

## [Building your own form](https://github.com/MonriPayments/monri-ios/wiki/Installation-Guide#building-your-own-form) <a href="#user-content-building-your-own-form" id="user-content-building-your-own-form"></a>

If you build your own payment form, you’ll need to collect at least your customers’ card numbers and expiration dates.
Monri strongly recommends collecting the CVC. You can optionally collect the user’s name and billing address for
additional fraud protection.

Once you’ve collected a customer’s information, you will need to exchange the information for a Monri token.

## [Creating & validating cards from a custom form](https://github.com/MonriPayments/monri-ios/wiki/Installation-Guide#creating--validating-cards-from-a-custom-form) <a href="#user-content-creating--validating-cards-from-a-custom-form" id="user-content-creating--validating-cards-from-a-custom-form"></a>

To create a Card object from data you’ve collected from other forms, you can create the object with its initializer.

```swift
let card = Card(number: "4111 1111 1111 1111", cvc: "123", expMonth: 10, expYear: 2022)

print("card.number valid = \(card.validateNumber())")
print("card.cvv valid = \(card.validateCVC())")
print("card.exp_date valid = \(card.validateExpiryDate())")
```

As you can see in the example above, the Card instance contains some helpers to validate that the card number passes the
Luhn check, that the expiration date is the future, and that the CVC looks valid. You’ll probably want to validate these
three things at once, so we’ve included a validateCard function that does so.

```swift
let card = Card(number: "4111 1111 1111 1111", cvc: "123", expMonth: 10, expYear: 2022)
if (!card.validateCard()) {
  // show errors
}
```

## [APIs](https://github.com/MonriPayments/monri-ios/wiki/Installation-Guide#apis) <a href="#user-content-apis" id="user-content-apis"></a>

iOS SDK supports:

* [Payment API](https://github.com/MonriPayments/monri-ios/wiki/Payment-API-Integration)
* [Tokens API](https://github.com/MonriPayments/monri-ios/wiki/Tokens-API-Integration)

Payment API is used to confirm payment directly in the app. This usually means that order's details are known at the
moment of collecting card's data.

Tokens API is used when your application collects card data to be used later in the order process.
