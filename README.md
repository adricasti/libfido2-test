# WebAuthn - libfido2 interoperability test

## libfido2 Input Format

The input of fido2-cred consists of base64 blobs and UTF-8 strings separated by newline characters ('\n').

When making a credential, fido2-cred expects its input to consist of:

1. client data hash (base64 blob);
2. relying party id (UTF-8 string);
3. user name (UTF-8 string);
4. user id (base64 blob).

So you would run it like this:

```
fido2-token -L // to get the list of devices
fido2-cred -M -d -i cred_param -o register_lib.txt ioreg://xxxxxxxx
```

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
fido2
```

## libfido2 Output Format

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

