[![Build Status](https://travis-ci.org/joebandenburg/libaxolotl-javascript.svg?branch=master)](https://travis-ci.org/joebandenburg/libaxolotl-javascript)

A JavaScript port of [libaxolotl](https://github.com/WhisperSystems/libaxolotl-android). Axolotl is a ratcheting forward
secrecy protocol that works in synchronous and asynchronous messaging environments. The protocol overview is available
[here](https://github.com/trevp/axolotl/wiki), and the details of the wire format are available
[here](https://github.com/WhisperSystems/TextSecure/wiki/ProtocolV2).

Currently this implementation only supports version 3 of the protocol.

## Installation

### Node.js

Install using npm:
```
$ npm install axolotl
```

and import using `"axolotl"`:
```javascript
var axolotl = require("axolotl");
```

### Browser

Install using bower:
```
$ bower install axolotl
```

and import using AMD:
```javascript
require(["axolotl"], function(axolotl) {

});
```

or without:
```javascript
window.axolotl(...)
```

## Getting started

### Dependencies

libaxolotl-javascript depends on [traceur-runtime](https://github.com/google/traceur-compiler) and
[protobuf.js](https://github.com/dcodeIO/ProtoBuf.js). These dependencies are not included in the distributed package.
If you installed libaxolotl-javascript using npm then there is nothing more you need to do - npm will download these
dependencies for you.

If you are using libaxolotl-javascript in the browser, you will have to provide the library's dependencies yourself. If
you're using AMD, then simply provide the location of these dependencies in your AMD configuration. Otherwise, include
the dependencies on the page before including `axolotl.js`.

### The Store interface

You need to provide an implementation of the `Store` interface when instantiating Axolotl. This is an object that
has the following methods.

*Note that all methods may return a Promise if the operation is asynchronous.*

#### getLocalIdentityKeyPair

```
getLocalIdentityKeyPair() → {KeyPair}
```

Get the local identity key pair created at install time. A key pair is a JavaScript object containing the keys `public`
and `private` which correspond to the public and private keys, respectively. These keys are of type ArrayBuffer.

#### getLocalRegistrationId

```
getLocalRegistrationId() → {Number}
```

Get the local registration identifier created at install time.

#### getLocalSignedPreKeyPair

```
getLocalSignedPreKeyPair(signedPreKeyId) → {KeyPair}
```

Get the local signed pre-key pair associated with the `signedPreKeyId`.

##### Parameters

Name|Type|Description
:---|:---|:----------
`signedPreKeyId`|Number|The identifier of the signed pre-key.

#### getLocalPreKeyPair

```
getLocalPreKeyPair(preKeyId) → {KeyPair}
```

Get the local pre-key pair associated with the `preKeyId`.

##### Parameters

Name|Type|Description
:---|:---|:----------
`preKeyId`|Number|The identifier of the pre-key.

#### getRemotePreKeyBundle

```
getRemotePreKeyBundle(remoteIdentity) → {PreKeyBundle}
```

Retrieve a pre-key bundle for a remote identity. This will likely be retrieved from a remote server. A `PreKeyBundle` is
a JavaScript object with the following properties:

Name|Type|Description
:---|:---|:----------
`identityKey`|ArrayBuffer|The remote identity's public key.
`preKeyId`|Number|The identifier of the pre-key included in this bundle.
`preKey`|ArrayBuffer|The public half of the pre-key.
`signedPreKeyId`|Number|The identifier of the signed pre-key included in this bundle.
`signedPreKey`|ArrayBuffer|The public half of the signed pre-key.
`signedPreKeySignature`|ArrayBuffer|The signature associated with the `signedPreKey`.

##### Parameters

Name|Type|Description
:---|:---|:----------
`remoteIdentity`|Identity|The identity of the remote entity. The type is the same as the type you pass to into
the methods on Axolotl.

#### isRemoteIdentityTrusted

```
isRemoteIdentityTrusted(remoteIdentity, identityPublicKey) → {Boolean}
```

Determine if the public key associated with the remote identity is trusted. It is up to the application to determine
an appropriate algorithm for this.

##### Parameters

Name|Type|Description
:---|:---|:----------
`remoteIdentity`|Identity|The identity of the remote entity.
`identityPublicKey`|ArrayBuffer|The purported remote identity's public key.

#### putRemoteIdentity

```
putRemoteIdentity(remoteIdentity, identityPublicKey) → {Void}
```

Store a public key to be associated with the remote identity. This is called when a session is successfully established
and gives the application an opportunity to register trust in the identity.

##### Parameters

Name|Type|Description
:---|:---|:----------
`remoteIdentity`|Identity|The identity of the remote entity.
`identityPublicKey`|ArrayBuffer|The remote identity's public key.

#### hasSession

```
hasSession(identity) → {Boolean}
```

Determine if there is a stored session for the identity.

##### Parameters

Name|Type|Description
:---|:---|:----------
`identity`|Identity|The identity.

#### getSession

```
getSession(identity) → {String}
```

Retrieve the session state associated with the identity.

##### Parameters

Name|Type|Description
:---|:---|:----------
`identity`|Identity|The identity.

#### putSession

```
putSession(identity, sessionState) → {Void}
```

Store the session state along with the identity.

##### Parameters

Name|Type|Description
:---|:---|:----------
`identity`|Identity|The identity.
`sessionState`|String|The serialised session state.

### The Crypto interface

You need to provide an implementation of the `Crypto` interface when instantiating Axolotl. This is an object that
has the following methods.

*Note that all methods may return a Promise if the operation is asynchronous.*

#### generateKeyPair

```
generateKeyPair() → {KeyPair}
```

Generate a fresh, random [Curve25519](http://en.wikipedia.org/wiki/Curve25519) public/private key pair suitable for use
with Diffie-Hellman key agreements. The returned private key should be an ArrayBuffer consisting of 32 bytes. The
returned public key should be an ArrayBuffer consisting of 33 bytes, where the first byte is equal to `0x05`.

#### calculateAgreement

```
calculateAgreement(theirPublicKey, ourPrivateKey) → {ArrayBuffer}
```

Compute a [Curve25519](http://en.wikipedia.org/wiki/Curve25519) Diffie-Hellman key agreement.

##### Parameters

Name|Type|Description
:---|:---|:----------
`theirPublicKey`|ArrayBuffer|Their 33 byte public key.
`ourPrivateKey`|ArrayBuffer|Our 32 byte private key.

#### randomBytes

```
randomBytes(byteCount) → {ArrayBuffer}
```

Generate `byteCount` bytes of cryptographically secure random data and return as an ArrayBuffer.

##### Parameters

Name|Type|Description
:---|:---|:----------
`byteCount`|Number|The number of bytes to generate.

#### sign

```
sign(privateKey, dataToSign) → {ArrayBuffer}
```

Produce an [Ed25519](http://en.wikipedia.org/wiki/EdDSA) signature. The returned signature should be an ArrayBuffer
consisting of 64 bytes.

##### Parameters

Name|Type|Description
:---|:---|:----------
`privateKey`|ArrayBuffer|The 32 byte private key to use to generate the signature.
`dataToSign`|ArrayBuffer|The data to be signed. May be any length.

#### verifySignature

```
verifySignature(publicKey, dataToSign, purportedSignature) → {Boolean}
```

Verify an [Ed25519](http://en.wikipedia.org/wiki/EdDSA) signature.

##### Parameters

Name|Type|Description
:---|:---|:----------
`privateKey`|ArrayBuffer|The 33 byte public half of the key used to produce the signature.
`dataToSign`|ArrayBuffer|The data that was signed. May be any length.
`purportedSignature`|ArrayBuffer|The purported signature to check.

#### hmac

```
hmac(key, data) → {ArrayBuffer}
```

Produce a HMAC-HASH using SHA-256. The returned ArrayBuffer should consist of 32 bytes.

##### Parameters

Name|Type|Description
:---|:---|:----------
`key`|ArrayBuffer|The mac key. May be any length.
`data`|ArrayBuffer|The data to be hashed. May be any length.

#### encrypt

```
encrypt(key, plaintext, iv) → {ArrayBuffer}
```

Encrypt the `plaintext` using AES-256-CBC.

##### Parameters

Name|Type|Description
:---|:---|:----------
`key`|ArrayBuffer|The 32 byte cipher key.
`plaintext`|ArrayBuffer|The data to be encrypted. May be any length.
`iv`|ArrayBuffer|A 16 byte random initialisation vector.

#### decrypt

```
decrypt(key, ciphertext, iv) → {ArrayBuffer}
```

Decrypt the `ciphertext` using AES-256-CBC.

##### Parameters

Name|Type|Description
:---|:---|:----------
`key`|ArrayBuffer|The 32 byte cipher key used to encrypt the data.
`ciphertext`|ArrayBuffer|The data to be decrypted. Should have a length that is a multiple of 16 bytes.
`iv`|ArrayBuffer|The 16 byte initialisation vector used to encrypt the data.

### Using Axolotl

Start by instantiating Axolotl:

```javascript
var axol = axolotl(crypto, store);
```

When your application is first installed, the client will likely need to register with the server. To do this, a number
of data needs to be generated:

```javascript
axol.generateIdentityKeyPair().then(...); // Generate our identity key
axol.generateRegistrationId().then(...); // Generate our registration id
axol.generatePreKeys(0, 100).then(...); // Generate the first set of our pre-keys to send to the server
axol.generateLastResortPreKey().then(...); // Generate our last restore pre-key to send to the server
axol.generateSignedPreKey(identityKeyPair, 1).then(...); // Generate our first signed pre-key to send to the server
```

Once registered, sending messages is very simple:

```javascript
var message = convertStringToBytes("Hello bob");
axol.encryptMessage("bob", message).then(function(ciphertext) {
    // Send ciphertext to alice
});
```

and on the receiving side:

```javascript
// When receiving a ciphertext, decide what type it is from the container and then decrypt
axol.decryptPreKeyWhisperMessage("alice", ciphertext).then(function(plaintext) {
    console.log(plaintext); // "Hello bob"
});
```

and that's it!