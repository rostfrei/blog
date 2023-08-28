# Setting up [Yubikey 5C NFC](https://www.yubico.com/si/product/yubikey-5-series/yubikey-5c-nfc/) GPG Github signing on MacOS

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

Export GPG private key.
`--armor` directs gpg to export in text readable form and not in binary form.
```
$ gpg --armor --export
-----BEGIN PGP PUBLIC KEY BLOCK-----

mDMEZOz+VBYJKwYBBAHaRw8BAQdAyrMiyiBa7XLauMlcdUTGhMUsGF7PN5M8XMDz
8NFNYze0IU1hcmtvIEt1a292ZWMgPGt1a292ZWNAZ21haWwuY29tPoiTBBMWCgA7
FiEEgje3VOS9zJMtZkBd5i09w9AC0bcFAmTs/lQCGwMFCwkIBwICIgIGFQoJCAsC
BBYCAwECHgcCF4AACgkQ5i09w9AC0bdzcgEAq2AAdmo+6c4vypEZD8aTyWSX1vXB
6REwSknJLZPD2TgBAPqtiD8pPfenOCqDD1Aglq9MFKJgUNhMaClQ4Wr2LEUPuDME
ZOz+VBYJKwYBBAHaRw8BAQdAXCwH0Vw6sbly/A1JtTe0i5lwhjznp7fPnyTsgcly
qW+IeAQYFgoAIBYhBII3t1TkvcyTLWZAXeYtPcPQAtG3BQJk7P5UAhsgAAoJEOYt
PcPQAtG34nEBAPO2ze81/NX1jq8Kw3E0g7kLKis02nMcRXgBJPLe0odjAP9i8R4U
50MXANZb+9e9aFBBJ2nCp0tFc6x2FoxMvifNDbg4BGTs/lQSCisGAQQBl1UBBQEB
B0B2y7JzWFN0hbO2WUq1Llq9cNUzE60F5sY7PUgamX3OfAMBCAeIeAQYFgoAIBYh
BII3t1TkvcyTLWZAXeYtPcPQAtG3BQJk7P5UAhsMAAoJEOYtPcPQAtG3Q0oA/3Lb
E7mLeULiX1cDcAkFWf8OW1RlBb7EMmyAPMiypd9YAQDAoKvyOam/xd69Z+p+nKAY
Tzk446vQfKqYCYbV4rT7AQ==
=DgRR
-----END PGP PUBLIC KEY BLOCK-----
```

Set public GPG key on Github under [your profile](https://github.com/settings/keys).

Setup `git` global config for gpg signing. 
```
$ git config --global user.signingkey 8237B754E4BDCC932D66405DE62D3DC3D002D1B7
$ git config --global commit.gpgsign true
```

We could also do this with `--edit` command
```
$ git config --global --edit
```
and default editor opens up with global config contens
```
[user]
        name = Marko Kukovec
        email = email@email.com
        signingkey = 8237B754E4BDCC932D66405DE62D3DC3D002D1B7
[core]
        excludesfile = /Users/markokukovec/.gitignore_global
[difftool "sourcetree"]
        cmd = opendiff \"$LOCAL\" \"$REMOTE\"
        path =
[mergetool "sourcetree"]
        cmd = /Applications/Sourcetree.app/Contents/Resources/opendiff-w.sh \"$LOCAL\" \"$REMOTE\" -ancestor \"$BASE\" -merge \"$MERGED\"
        trustExitCode = true
[commit]
        template = /Users/markokukovec/.stCommitMsg
        gpgsign = true
```

We are ready for Github GPG signing!
