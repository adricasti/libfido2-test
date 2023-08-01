# WebAuthn - libfido2 interoperability test

[WebAuthn](https://www.w3.org/TR/webauthn-2/) is a standard for strong authentication on the Web, it's available in all major browsers. There are also native APIs available for [Android](https://developers.google.com/identity/fido/android/native-apps) and [Windows](https://github.com/microsoft/webauthn). Behind the scenes, WebAuthn communicates with FIDO authenticators using the [CTAP2](https://fidoalliance.org/specs/fido-v2.1-ps-20210615/fido-client-to-authenticator-protocol-v2.1-ps-errata-20220621.html) protocol.

[libfido2](https://developers.yubico.com/libfido2/) is an alternative library for destop operating systems that provides utilities to interact with FIDO2 devices.

# Registration through the browser

To register a credential in the browser with `hmac-secret` extension you would need to specify that in the extension for the key creation, like this:

```javascript
  const publicKey = {
    /* ... */
    extensions: {
      hmacCreateSecret: true,
    },
  };

  navigator.credentials.create({ publicKey })
```

# Assertion through the library

When obtaining an assertion, **fido2-assert** expects its input to consist of:

1. client data hash (base64 blob)
2. relying party id (UTF-8 string)
3. credential id, if credential not resident (base64 blob)
4. hmac salt, if the FIDO2 hmac-secret extension is enabled (base64 blob)

Using the response from the browser registration, you can build that file and then try:

```
fido2-token -L // to list the available tokens
fido2-assert -G -h -i assert_param ioreg://xxxxxxxxxx
```
