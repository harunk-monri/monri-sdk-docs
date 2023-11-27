# Tokens API Integration

Monri has created a Java library for Android, allowing you to easily submit payments from an Android app. With our
mobile library, we address PCI compliance by eliminating the need to send card data directly to your server. Instead,
our libraries send the card data directly to our servers, where we can convert them
to [tokens](https://monri.com/docs/api#tokens).

Your app will receive the token back, and can then send the token to an endpoint on your server, where it can be used to
process a payment.

### Using tokens

Using the payment token, however it was obtained, requires an API call from your server using your secret merchant
key. (For security purposes, you should never embed your secret merchant key in your app.)

Set up an endpoint on your server that can receive an HTTP POST call for the token. In the `onSuccess` callback (when
using your own form), you’ll need to POST the supplied token to your server. Make sure any communication with your
server is [SSL secured](https://monri.com/docs/security) to prevent eavesdropping.

### Building your own form

If you build your own payment form, you’ll need to collect at least your customers’ card numbers and expiration dates.
Monri strongly recommends collecting the CVC. You can optionally collect the user’s name and billing address for
additional fraud protection.

Once you’ve collected a customer’s information, you will need to exchange the information for a Monri token.

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