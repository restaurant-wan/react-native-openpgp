React-Native-OpenPGP
==========

[React-Native-OpenPGP](http://openpgpjs.org/) is a Javascript implementation of the OpenPGP protocol based on [OpenPGP.js](https://github.com/openpgpjs/openpgpjs).


## Getting started

#### Installation

    npm install --save react-native-openpgp
    react-native link react-native-openpgp

Note: Run `npm install -g rnpm` if you haven't installed RNPM (React-Native Package Manager) yet!
Alternatively you can add the Android and iOS modules library by following the official guide.

## Usage

```js
import * as openpgp from 'react-native-openpgp';
```

#### Encrypt and decrypt *String* data with a password

```js
var options, encrypted;

options = {
  data: 'Hello, World!',      // input as String
  passwords: ['secret stuff'] // multiple passwords possible
};

// Because of the way the library works (random values have to be generated on natively!),
// it is always highly recommended to call this method before doing any actual work!
openpgp.prepareRandomValues()
  .then(() => {
    openpgp.encrypt(options)
      .then((ciphertext) => {
        encrypted = ciphertext.data; // '-----BEGIN PGP MESSAGE ... END PGP MESSAGE-----'
      })
      .catch((error) => {
        console.log("Something went wrong: " + error);
      });
  });
```

```js
options = {
  message: openpgp.readMessage(encrypted), // parse armored message
  password: 'secret stuff'                         // decrypt with password
};

// Because of the way the library works (random values have to be generated natively!),
// it is always highly recommended to call this method before doing any actual work!
openpgp.prepareRandomValues()
  .then(() => {
    openpgp.decrypt(options)
      .then((plaintext) => {
        return plaintext.data; // 'Hello, World!'
      })
      .catch((error) => {
        console.log("Something went wrong: " + error);
      });
  });
```

#### Encrypt and decrypt *Uint8Array* data with PGP keys

```js
var options, encrypted;

var pubkey = '-----BEGIN PGP PUBLIC KEY BLOCK ... END PGP PUBLIC KEY BLOCK-----';
var privkey = '-----BEGIN PGP PRIVATE KEY BLOCK ... END PGP PRIVATE KEY BLOCK-----';

options = {
  data: new Uint8Array([0x01, 0x01, 0x01]),           // input as Uint8Array
  publicKeys: openpgp.readArmoredKey(pubkey).keys,   // for encryption
  privateKeys: openpgp.readArmoredKey(privkey).keys, // for signing (optional)
  armor: false                                        // don't ASCII armor
};

// Because of the way the library works (random values have to be generated natively!),
// it is always highly recommended to call this method before doing any actual work!
openpgp.prepareRandomValues()
  .then(() => {
    openpgp.encrypt(options)
      .then((ciphertext) => {
        encrypted = ciphertext.message.packets.write(); // get raw encrypted packets as Uint8Array
      })
      .catch((error) => {
        console.log("Something went wrong: " + error);
      });
  });
```

```js
options = {
  message: openpgp.readBinaryMessage(encrypted),             // parse encrypted bytes
  publicKeys: openpgp.readArmoredKey(pubkey).keys,     // for verification (optional)
  privateKey: openpgp.readArmoredKey(privkey).keys[0], // for decryption
  format: 'binary'                                      // output as Uint8Array
};

// Because of the way the library works (random values have to be generated natively!),
// it is always highly recommended to call this method before doing any actual work!
openpgp.prepareRandomValues()
  .then(() => {
    openpgp.decrypt(options)
      .then((plaintext) => {
        return plaintext.data // Uint8Array([0x01, 0x01, 0x01])
      })
      .catch((error) => {
        console.log("Something went wrong: " + error);
      });
  });
```

#### Generate new key pair

```js
var options = {
  userIds: [{ name:'Jon Smith', email:'jon@example.com' }], // multiple user IDs
  numBits: 2048,                                            // RSA key size
  passphrase: 'super long and hard to guess secret'         // protects the private key
};

// Because of the way the library works (random values have to be generated natively!),
// it is always highly recommended to call this method before doing any actual work!
openpgp.prepareRandomValues()
  .then(() => {
    openpgp.generateKey(options)
      .then((key) => {
        var privkey = key.privateKeyArmored; // '-----BEGIN PGP PRIVATE KEY BLOCK ... '
        var pubkey = key.publicKeyArmored;   // '-----BEGIN PGP PUBLIC KEY BLOCK ... '
      })
      .catch((error) => {
        console.log("Something went wrong: " + error);
      });
  });

```
