# Setting up [Yubikey 5C NFC](https://www.yubico.com/si/product/yubikey-5-series/yubikey-5c-nfc/) GPG application on MacOS

Install needed packages
```
$ brew install ykman yubico-piv-tool gnupg pinentry-mac
```

Insert key into USB C slot and start editing `card`
```
$ gpg --card-edit

gpg: WARNING: server 'gpg-agent' is older than us (2.2.41 < 2.4.3)
gpg: Note: Outdated servers may lack important security fixes.
gpg: Note: Use the command "gpgconf --kill all" to restart them.
gpg: WARNING: server 'scdaemon' is older than us (2.2.41 < 2.4.3)
gpg: Note: Outdated servers may lack important security fixes.
gpg: Note: Use the command "gpgconf --kill all" to restart them.
Reader ...........: Yubico YubiKey OTP FIDO CCID
Application ID ...: D2760001240103040006234077060000
Application type .: OpenPGP
Version ..........: 3.4
Manufacturer .....: Yubico
Serial number ....: 23407706
Name of cardholder: [not set]
Language prefs ...: [not set]
Salutation .......:
URL of public key : [not set]
Login data .......: [not set]
Signature PIN ....: not forced
Key attributes ...: rsa2048 rsa2048 rsa2048
Max. PIN lengths .: 127 127 127
PIN retry counter : 3 0 3
Signature counter : 0
KDF setting ......: off
UIF setting ......: Sign=off Decrypt=off Auth=off
Signature key ....: [none]
Encryption key....: [none]
Authentication key: [none]
General key info..: [none]
```

Factory reset GPG application. It will not affect PIV or other applications. First we have to enable admin commands.
```
gpg/card> admin
Admin commands are allowed
```
All awailable commands
```
gpg/card>
admin          factory-reset  forcesig       help           key-attr       list           name           passwd         salutation     unblock        verify
cafpr          fetch          generate       kdf-setup      lang           login          openpgp        quit           uif            url
```
Factory reset
```
gpg/card> factory-reset
gpg: OpenPGP card no. D2760001240103040006234077060000 detected

gpg: Note: This command destroys all keys stored on the card!

Continue? (y/N) y
Really do a factory reset? (enter "yes") yes
```
Exit application with `Ctrl+D` and reenter editing card.
```
$ gpg --card-edit
```
Enter admin mode
```
gpg/card> admin
Admin commands are allowed
```

Factory default PIN numbers
```
User PIN: 123456
Admin PIN: 12345678
```

Now we have clean factory reset GPG application Yubikey.

First let's enable `KDF` (Key Derivation Function). Read more about reasons for KDF [here](https://peterbabic.dev/blog/openpgp-smartcard-kdf-issue-bad-pin/). Essentially it means that pin numbers you type in are hashed before sending them to the card. Pins on the card are also stored in hashed form. For PIN check to be valid, card checks if hashes are the same.
We have to do this step first! If we do it later, we will lockout ourserlves out of the card access as pins from clear form are not rehashed and stored when KDF is enabled.

Enable KDF
```
gpg/card> kdf-setup
```
Prompt pops up requesting `Admin PIN`. Admin PIN at this stage is still `12345678`

Change PIN numbers. At this stage `User PIN` is still `123456` and `Admin PIN` is still `12345678`
```
gpg/card> passwd
gpg: OpenPGP card no. D2760001240103040006234077060000 detected

1 - change PIN
2 - unblock PIN
3 - change Admin PIN
4 - set the Reset Code
Q - quit

Your selection? 1
PIN changed.

1 - change PIN
2 - unblock PIN
3 - change Admin PIN
4 - set the Reset Code
Q - quit

Your selection? 3
Error changing the PIN: Bad PIN

1 - change PIN
2 - unblock PIN
3 - change Admin PIN
4 - set the Reset Code
Q - quit
```

Let's configure some card owner data.

Configure name
```
gpg/card> name
Cardholder's surname: Kukovec
Cardholder's given name: Marko
```

Configure salutation
```
gpg/card> salutation
Salutation (M = Mr., F = Ms., or space): m
```

Configure key types card will generate. We want to use modern ECC (Elliptic Curve Cryptography). Ed25519 is a specific implementation of ECC that uses the Edwards Curve 255192. It is commonly used for digital signing and offers constant-time performance2. The EdDSA (Edwards Digital Signing Algorithm) is the digital signing algorithm used in Ed25519.
```
gpg/card> key-attr
Changing card key attribute for: Signature key
Please select what kind of key you want:
   (1) RSA
   (2) ECC
Your selection? 2
Please select which elliptic curve you want:
   (1) Curve 25519 *default*
   (4) NIST P-384
   (6) Brainpool P-256
Your selection? 1
The card will now be re-configured to generate a key of type: ed25519
Note: There is no guarantee that the card supports the requested
      key type or size.  If the key generation does not succeed,
      please check the documentation of your card to see which
      key types and sizes are supported.
Changing card key attribute for: Encryption key
Please select what kind of key you want:
   (1) RSA
   (2) ECC
Your selection? 2
Please select which elliptic curve you want:
   (1) Curve 25519 *default*
   (4) NIST P-384
   (6) Brainpool P-256
Your selection? 1
The card will now be re-configured to generate a key of type: cv25519
Changing card key attribute for: Authentication key
Please select what kind of key you want:
   (1) RSA
   (2) ECC
Your selection? 2
Please select which elliptic curve you want:
   (1) Curve 25519 *default*
   (4) NIST P-384
   (6) Brainpool P-256
Your selection? 1
The card will now be re-configured to generate a key of type: ed25519
```
We'll have to type in `Admin PIN` for every key change. Signature, Encryption and Authentication keys.

Force signing with PIN
```
gpg/card> forcesig
```

Let's check current configuration status
```
gpg/card> list

Reader ...........: Yubico YubiKey OTP FIDO CCID
Application ID ...: D2760001240103040006234077060000
Application type .: OpenPGP
Version ..........: 3.4
Manufacturer .....: Yubico
Serial number ....: 23407706
Name of cardholder: Marko Kukovec
Language prefs ...: [not set]
Salutation .......: Mr.
URL of public key : [not set]
Login data .......: [not set]
Signature PIN ....: forced
Key attributes ...: ed25519 cv25519 ed25519
Max. PIN lengths .: 127 127 127
PIN retry counter : 3 0 3
Signature counter : 0
KDF setting ......: on
UIF setting ......: Sign=off Decrypt=off Auth=off
Signature key ....: [none]
Encryption key....: [none]
Authentication key: [none]
General key info..: [none]
```

Generate keys. In this approach we don't generate keys on the computer and then transfer them to the card. In this approach card itself generate GPG keys and store them internaly. Private key never leaves the card!
```
gpg/card> generate
Make off-card backup of encryption key? (Y/n) n
Please specify how long the key should be valid.
         0 = key does not expire
      <n>  = key expires in n days
      <n>w = key expires in n weeks
      <n>m = key expires in n months
      <n>y = key expires in n years
Key is valid for? (0) 0
Key does not expire at all
Is this correct? (y/N) y

GnuPG needs to construct a user ID to identify your key.

Real name: Marko Kukovec
Email address: email@email.com
Comment:
You selected this USER-ID:
    "Marko Kukovec <email@email.com>"

Change (N)ame, (C)omment, (E)mail or (O)kay/(Q)uit? o
gpg: directory '/Users/markokukovec/.gnupg/openpgp-revocs.d' created
gpg: revocation certificate stored as '/Users/markokukovec/.gnupg/openpgp-revocs.d/8237B754E4BDCC932D66405DE62D3DC3D002D1B7.rev'
public and secret key created and signed.
```

Let's check current configuration status
```
gpg/card> list

Reader ...........: Yubico YubiKey OTP FIDO CCID
Application ID ...: D2760001240103040006234077060000
Application type .: OpenPGP
Version ..........: 3.4
Manufacturer .....: Yubico
Serial number ....: 23407706
Name of cardholder: Marko Kukovec
Language prefs ...: [not set]
Salutation .......: Mr.
URL of public key : [not set]
Login data .......: [not set]
Signature PIN ....: forced
Key attributes ...: ed25519 cv25519 ed25519
Max. PIN lengths .: 127 127 127
PIN retry counter : 3 0 3
Signature counter : 4
KDF setting ......: on
UIF setting ......: Sign=off Decrypt=off Auth=off
Signature key ....: 8237 B754 E4BD CC93 2D66  405D E62D 3DC3 D002 D1B7
      created ....: 2023-08-28 20:06:44
Encryption key....: 855E D36B BAA0 6D0A 9F0B  0B8D 945D 7D68 528C 4C50
      created ....: 2023-08-28 20:06:44
Authentication key: D66A C61B A883 7F1D D8BC  B0E9 D5F1 C57C 0895 2972
      created ....: 2023-08-28 20:06:44
General key info..:
pub  ed25519/E62D3DC3D002D1B7 2023-08-28 Marko Kukovec <email@email.com>
sec>  ed25519/E62D3DC3D002D1B7  created: 2023-08-28  expires: never
                                card-no: 0006 23407706
ssb>  ed25519/D5F1C57C08952972  created: 2023-08-28  expires: never
                                card-no: 0006 23407706
ssb>  cv25519/945D7D68528C4C50  created: 2023-08-28  expires: never
                                card-no: 0006 23407706
```
From the output we can see that short version of the key id is `E62D3DC3D002D1B7`

Exit application with `Ctrl+D` 

List secret keys with GPG application.
```
$ gpg --list-secret-keys
gpg: WARNING: server 'gpg-agent' is older than us (2.2.41 < 2.4.3)
gpg: Note: Outdated servers may lack important security fixes.
gpg: Note: Use the command "gpgconf --kill all" to restart them.
gpg: problem with fast path key listing: IPC parameter error - ignored
[keyboxd]
---------
sec>  ed25519 2023-08-28 [SC]
      8237B754E4BDCC932D66405DE62D3DC3D002D1B7
      Card serial no. = 0006 23407706
uid           [ unknown] Marko Kukovec <email@email.com>
ssb>  ed25519 2023-08-28 [A]
ssb>  cv25519 2023-08-28 [E]
```
From the output we can see that long version of the key id is `8237B754E4BDCC932D66405DE62D3DC3D002D1B7`




## References
[YubiKey Technical Manual](https://docs.yubico.com/hardware/yubikey/yk-tech-manual/index.html#yubikey-technical-manual)

[YubiKey Manager (ykman) CLI and GUI Guide](https://docs.yubico.com/software/yubikey/tools/ykman/)

[Yubico PIV Tool](https://developers.yubico.com/yubico-piv-tool/)

[Securing SSH with the YubiKey](https://developers.yubico.com/SSH/)

[What is PIV](https://developers.yubico.com/PIV/)

[What is PGP](https://developers.yubico.com/PGP/)