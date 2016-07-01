React-Native-OpenPGP
==========

[React-Native-OpenPGP](http://openpgpjs.org/) is a Javascript implementation of the OpenPGP protocol based on [OpenPGP.js](https://github.com/openpgpjs/openpgpjs).


### Getting started

#### npm

    npm install --save github:orhan/react-native-openpgp
    rnpm link react-native-openpgp

Note: Run `npm install -g rnpm` if you haven't installed RNPM (React-Native Package Manager) yet!

### Usage

```js
var openpgp = require('react-native-openpgp');
```

#### Before Usage

Before using any of the methods below, it is ~~highly recommended~~ **necessary** to generate some random values to seed the randomization algorithm of this library. You don't have to do this for every operation, but you are highly advised to do so.

Basic example:
```js
openpgp.prepareRandomValues()
  .then(() => {

    // YOUR CALLS START HERE!
    openpgp.encrypt(options).then(function(ciphertext) {
        encrypted = ciphertext.data;
    });

  })
```

#### Encrypt and decrypt *String* data with a password

```js
var options, encrypted;

options = {
    data: 'Hello, World!',      // input as String
    passwords: ['secret stuff'] // multiple passwords possible
};

openpgp.encrypt(options).then(function(ciphertext) {
    encrypted = ciphertext.data; // '-----BEGIN PGP MESSAGE ... END PGP MESSAGE-----'
});
```

```js
options = {
    message: openpgp.readMessage(encrypted), // parse armored message
    password: 'secret stuff'                         // decrypt with password
};

openpgp.decrypt(options).then(function(plaintext) {
    return plaintext.data; // 'Hello, World!'
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

openpgp.encrypt(options).then(function(ciphertext) {
    encrypted = ciphertext.message.packets.write(); // get raw encrypted packets as Uint8Array
});
```

```js
options = {
    message: openpgp.readBinaryMessage(encrypted),             // parse encrypted bytes
    publicKeys: openpgp.readArmoredKey(pubkey).keys,     // for verification (optional)
    privateKey: openpgp.readArmoredKey(privkey).keys[0], // for decryption
    format: 'binary'                                      // output as Uint8Array
};

openpgp.decrypt(options).then(function(plaintext) {
    return plaintext.data // Uint8Array([0x01, 0x01, 0x01])
});
```

#### Generate new key pair

```js
var options = {
    userIds: [{ name:'Jon Smith', email:'jon@example.com' }], // multiple user IDs
    numBits: 2048,                                            // RSA key size
    passphrase: 'super long and hard to guess secret'         // protects the private key
};

openpgp.generateKey(options).then(function(key) {
    var privkey = key.privateKeyArmored; // '-----BEGIN PGP PRIVATE KEY BLOCK ... '
    var pubkey = key.publicKeyArmored;   // '-----BEGIN PGP PUBLIC KEY BLOCK ... '
});
```
