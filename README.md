# WebAuthn - libfido2 interoperability test

[WebAuthn](https://www.w3.org/TR/webauthn-2/) is a standard for strong authentication on the Web, it's available in all major browsers. There are also native APIs available for [Android](https://developers.google.com/identity/fido/android/native-apps) and [Windows](https://github.com/microsoft/webauthn). Behind the scenes, WebAuthn communicates with FIDO authenticators using the [CTAP2](https://fidoalliance.org/specs/fido-v2.1-ps-20210615/fido-client-to-authenticator-protocol-v2.1-ps-errata-20220621.html) protocol.

[libfido2](https://developers.yubico.com/libfido2/) is an alternative library for destop operating systems that provides utilities to interact with FIDO2 devices.

# libfido2 Command Line Tools

## fido2-cred Input Format

The input of fido2-cred consists of base64 blobs and UTF-8 strings separated by newline characters ('\n').

When making a credential, fido2-cred expects its input to consist of:

1. client data hash (base64 blob);
2. relying party id (UTF-8 string);
3. user name (UTF-8 string);
4. user id (base64 blob).

So you would run it like this: (don't forget to confirm user presence by touching the button of a USB security key or tapping the card in the reader)

```
fido2-token -L // to get the list of devices
fido2-cred -M -i cred_param -o register_lib.txt ioreg://xxxxxxxx
```

Optionally you can use:

`-d` to include debug information
`-h` to use the `hmac-secret` extension

When verifying a credential, fido2-cred expects its input to consist of:

1. client data hash (base64 blob)
2. relying party id (UTF-8 string)
3. credential format (UTF-8 string)
4. authenticator data (base64 blob)
5. credential id (base64 blob)
6. attestation signature (base64 blob)
7. attestation certificate (optional, base64 blob).

UTF-8 strings passed to fido2-cred must not contain embedded newline or NUL characters.

You can verify the above created credential like this:

```
fido2-cred -V -i register_lib.txt
```

## fido2-cred Output Format

The output of fido2-cred consists of base64 blobs, UTF-8 strings, and PEM-encoded public keys separated by newline characters ('\n').

Upon the successful generation of a credential, fido2-cred outputs:

1. client data hash (base64 blob)
2. relying party id (UTF-8 string)
3. credential format (UTF-8 string)
4. authenticator data (base64 blob)
5. credential id (base64 blob)
6. attestation signature (base64 blob)
7. attestation certificate, if present (base64 blob).
8. the credential's associated 32-byte symmetric key (“largeBlobKey”), if present (base64 blob).

Upon the successful verification of a credential, fido2-cred outputs:

1. credential id (base64 blob);
2. PEM-encoded credential key.

## fido2-assert Input Format

The input of fido2-assert consists of base64 blobs and UTF-8 strings separated by newline characters ('\n').

When obtaining an assertion, fido2-assert expects its input to consist of:

1. client data hash (base64 blob);
2. relying party id (UTF-8 string);
3. credential id, if credential not resident (base64 blob);
4. hmac salt, if the FIDO2 hmac-secret extension is enabled (base64 blob);

When verifying an assertion, fido2-assert expects its input to consist of:

1. client data hash (base64 blob);
2. relying party id (UTF-8 string);
3. authenticator data (base64 blob);
4. assertion signature (base64 blob);
5. UTF-8 strings passed to fido2-assert must not contain embedded newline or NUL characters.

## fido2-assert Output Format

The output of fido2-assert consists of base64 blobs and UTF-8 strings separated by newline characters ('\n').

For each generated assertion, fido2-assert outputs:

1. client data hash (base64 blob);
2. relying party id (UTF-8 string);
3. authenticator data (base64 blob);
4. assertion signature (base64 blob);
5. user id, if credential resident (base64 blob);
6. hmac secret, if the FIDO2 hmac-secret extension is enabled (base64 blob);
7. the credential's associated 32-byte symmetric key (“largeBlobKey”), if requested (base64 blob).

When verifying an assertion, fido2-assert produces no output.

EXAMPLES

Assuming cred contains a es256 credential created according to the steps outlined in fido2-cred(1), obtain an assertion from an authenticator at /dev/hidraw5 and verify it:
$ echo assertion challenge | openssl sha256 -binary | base64 > assert_param
$ echo relying party >> assert_param
$ head -1 cred >> assert_param
$ tail -n +2 cred > pubkey
$ fido2-assert -G -i assert_param /dev/hidraw5 | fido2-assert -V pubkey es25