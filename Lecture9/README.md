# Exercises 9: Security and Cryptography

1. **Entropy.**

   1. Suppose a password is chosen as a concatenation of five lower-case
      dictionary words, where each word is selected uniformly at random from a
      dictionary of size 100,000. An example of such a password is
      `correcthorsebatterystaple`. How many bits of entropy does this have?

      The entropy is equal to `log_2(# of possibilities)`. The total number of possibilities would be `100,000 ** 5` (100,000 raised to the power of 5). Then the entropy equals approximately **83 bits**.

   1. Consider an alternative scheme where a password is chosen as a sequence
      of 8 random alphanumeric characters (including both lower-case and
      upper-case letters). An example is `rg8Ql34g`. How many bits of entropy
      does this have?

      For an alphanumeric character, there are in total `26+26+10=62` possibilities. Since there are 8 random characters, the total number of possibilities would be `62 ** 8`. Ten the entropy equals approximately **48 bits**.

   1. Which is the stronger password?

      The first one is stronger since the entropy is higher.

   1. Suppose an attacker can try guessing 10,000 passwords per second. On
      average, how long will it take to break each of the passwords?

      In a year, the attacker can try guessing `365*24*3600*10000=315360000000` passwords. For the first password, it takes approximately `(2**83)/315360000000=3*10^13` years. For the second password, it takes approximately `(2**48)/315360000000=893` years.

1. **Cryptographic hash functions.** Download a Debian image from a
   [mirror](https://www.debian.org/CD/http-ftp/) (e.g. [from this Argentinean
   mirror](http://debian.xfree.com.ar/debian-cd/current/amd64/iso-cd/).
   Cross-check the hash (e.g. using the `sha256sum` command) with the hash
   retrieved from the official Debian site (e.g. [this
   file](https://cdimage.debian.org/debian-cd/current/amd64/iso-cd/SHA256SUMS)
   hosted at `debian.org`, if you've downloaded the linked file from the
   Argentinean mirror).

   First download the OS appropriate `netinst` file from the mirror site using `curl` command. (Mine was macOS so downloaded the mac version.) Also download the `SHA256SUMS` file.

   ```
   $ curl -O -L -C - http://debian.xfree.com.ar/debian-cd/current/amd64/iso-cd/debian-mac-10.4.0-amd64-netinst.iso
   $ curl -O https://cdimage.debian.org/debian-cd/current/amd64/iso-cd/SHA256SUMS
   ```

   Then extract only the line that contains the hash of the mac version file from the `SHA256SUMS` file. Then use the `shasum` command with a `--check` tag. The command will search the directory and see if there is a file that matches the extracted hash list i.e hash for the mac. (Note `sha256` command is not available on macOS, but can be replicated as `shasum -a 256`.)

   ```
   $ sed '/mac/!d' SHA256SUMS | shasum --check
   debian-mac-10.4.0-amd64-netinst.iso: OK
   ```

   The hash of the file downloaded from the mirror matches the official list of hashes, so the file should be a correct one.

1. **Symmetric cryptography.** Encrypt a file with AES encryption, using
   [OpenSSL](https://www.openssl.org/): `openssl aes-256-cbc -salt -in {input filename} -out {output filename}`. Look at the contents using `cat` or
   `hexdump`. Decrypt it with `openssl aes-256-cbc -d -in {input filename} -out {output filename}` and confirm that the contents match the original using
   `cmp`.

   Create a file. Enter the message and save it by `Ctrl+D`.

   ```
   $ cat > secret.text
   hello secret world
   ```

   Encrypt the file in `aes-256`. I used `password` for password just for illustration purposes. Then decrypt and confirm that the contents match. The contents do match.

   ```
   $ openssl aes-256-cbc -salt -in secret.txt -out secret.enc.txt
   enter aes-256-cbc encryption password:
   Verifying - enter aes-256-cbc encryption password:
   $ openssl aes-256-cbc -d -in secret.enc.txt -out secret.dec.txt
   enter aes-256-cbc decryption password:
   $ cmp secret.txt secret.dec.txt | echo $?
   0
   ```

1. **Asymmetric cryptography.**

   1. Set up [SSH
      keys](https://www.digitalocean.com/community/tutorials/how-to-set-up-ssh-keys--2)
      on a computer you have access to (not Athena, because Kerberos interacts
      weirdly with SSH keys). Rather than using RSA keys as in the linked
      tutorial, use more secure [ED25519
      keys](https://wiki.archlinux.org/index.php/SSH_keys#Ed25519). Make sure
      your private key is encrypted with a passphrase, so it is protected at
      rest.

      Ed25519 key pairs can be generated with:

      ```
      $ ssh-keygen -t ed25519
      ```

   1. [Set up GPG](https://www.digitalocean.com/community/tutorials/how-to-use-gpg-to-encrypt-and-sign-messages)

      ```
      $ brew install gpg
      $ gpg --gen-key
      $ gpg --output ~/revocation.crt --gen-revoke your_email@address.com
      $ chmod 600 ~/revocation.crt
      ```

   1. Send Anish an encrypted email ([public key](https://keybase.io/anish)).

      ```
      gpg --encrypt --sign --armor -r person@email.com name_of_file
      ```

      `person@email.com` denotes the email address of the receiver. `name_of_file` denotes the message.

   1. Sign a Git commit with `git commit -C` or create a signed Git tag with
      `git tag -s`. Verify the signature on the commit with `git show --show-signature` or on the tag with `git tag -v`.

      `git commit --help`:

      ```
      -C <commit>, --reuse-message=<commit>
           Take an existing commit object, and reuse the log message and the authorship information (including the timestamp) when creating the commit.
      ```

      Seems `git commit -S` is the way to sign a Git commit. Is `git commit -C` a mistake? I do not know of yet.

      Edit: I rasied an issue on the official MIT Missing Sem repo and got affirmed it was a typo indeed. [My issue](https://github.com/missing-semester/missing-semester/issues/56)

      [How](https://help.github.com/en/github/authenticating-to-github/about-commit-signature-verification#gpg-commit-signature-verification) to do GPG commit signature verification. Also a [link](https://stackoverflow.com/questions/41052538/git-error-gpg-failed-to-sign-data/41054093#41054093) I consulted to troubleshoot `error: gpg failed to sign the data`. Sign a git commit and then verify the signature.

      ```
      $ git commit -S -m  "signed commit test"
      [master 1e7b6ad] signed commit test
      1 file changed, 1 insertion(+), 1 deletion(-)
      $ git show --show-signature
      commit 1e7b6ad5971826014e1bad3ec2c82ae282ae763f (HEAD -> master)
      gpg: Signature made Mon  1 Jun 00:10:47 2020 KST
      gpg:                using RSA key ***
      gpg: Good signature from "Woonha Kim <kwh051@gmail.com>" [ultimate]
      ```
