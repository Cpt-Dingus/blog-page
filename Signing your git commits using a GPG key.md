To sign your commits via git, you will need to create and register a GPG key. I have personally found it a bit tedious to set up, since it requires like 4 separate doc pages. This file will sum it up into one place.

# Creating a GPG signing key

- To create an RSA4096 key: install gnupg and run `gpg --default-new-key-algo rsa4096 --gen-key`

If you want to create a more granular key (Not RSA4096), use the `gpg --full-generate-key`command instead

> **DISCLAIMER**: GPG will ask you for your name and email address, __these will be listed in every commit you create.__

> If you want to hide your real email, **use the noreply address** found by going to Github.com > Your profile > Settings > Email, it will be cited in both the `Primary email address` field as well as below the `Keep my email address private` checkbox. For example my address is 100243410+Cpt-Dingus@users.noreply.github.com


# Adding the GPG key to your github account

Now that we have generated the GPG key we'll be using, we need to export its contents in order to add it to our github account.
#### 1. Get the key ID
- Run `gpg --list-secret-keys --keyid-format=long` and copy the value after `4096R/` in the `sec` line

Sample output: 

```bash
$ gpg --list-secret-keys --keyid-format=long
/Users/hubot/.gnupg/secring.gpg
------------------------------------
sec   4096R/3AA5C34371567BD2 2016-03-10 [expires: 2017-03-10]
uid                          Hubot <hubot@example.com>
ssb   4096R/4BB6D45482678BE3 2016-03-10
```

In this case the ID would be `3AA5C34371567BD2`

#### 2. Add the key to your account

- Run `gpg --armor --export <key-id>`, this will print the public key to your console. Copy the whole output, including the `BEGIN PGP KEY` and `END PGP KEY` lines.
- Navigate to Github.com > Your profile > Settings > SSH and GPG keys > New GPG key > Paste the contents into the `Key` field 


# Configuring git to use the GPG key

1. Register the key by running `git config --global user.signingkey <key-id>`
2. Set your username and email accordingly by running `git config --global user.name <name>` and `git config --global user.email <email>` with the same credentials you provided to the key
3. Choose how you want to sign the commits:
- Make all your commits be signed automatically by running `git config --global commit.gpgsign true`
- Manually sign commits by passing the `-S` flag to the `git commit` command

You are now able to create commits with a verified signature.
