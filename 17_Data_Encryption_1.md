# 17 Data Encryption. Part 1
## Asymmetrical Encryption ##
A matched pair of keys exist A and B. Either key can be use to encrypt or decrypt a message. However, if a message is encrypted with A it can only be decrypted with B and vice versa. Now suppose A is a public key that you share with others and B is your personal private key. This allows two possibilities:

1. If a message is encrypted using your public key B then it can only be decrypted by you (using your private key A). This allows other to send you private messages and allows you to encrypt files that you can decrypt using your private key.

2. If a message is encrypted using your private key A then it can only be decrypted using your public key B. This allows others to verify the authenticity of your message (i.e., confirm that it came from you). You sign the message with your private key and recipients can verify that it is from you using your public key.

## GNU Privacy Guard (GPG) ##

GPG is free asymmetrical encryption software developed by the GNU/FSF that can be used for maintaining privacy/security on computers and networks. According to the gpg man page:

> gpg is the OpenPGP (Pretty Good Privacy) part of the GNU Privacy Guard (GnuPG). It is a tool to provide digital encryption and signing services using the OpenPGP standard.

### Creating GPG keys ###

The first step in using gpg is to generate a key pair. A simple way to do this is to use the `gpg --quick-generate-key` command. The syntax is

`gpg --full-generate-key`

Here is an example:

```
dante@home$ gpg --full-generate-key
gpg (GnuPG) 2.2.19; Copyright (C) 2019 Free Software Foundation, Inc.
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.

Please select what kind of key you want:
   (1) RSA and RSA (default)
   (2) DSA and Elgamal
   (3) DSA (sign only)
   (4) RSA (sign only)
  (14) Existing key from card
Your selection? 1
RSA keys may be between 1024 and 4096 bits long.
What keysize do you want? (3072) 4096
Requested keysize is 4096 bits
Please specify how long the key should be valid.
         0 = key does not expire
      <n>  = key expires in n days
      <n>w = key expires in n weeks
      <n>m = key expires in n months
      <n>y = key expires in n years
Key is valid for? (0) 1w
Key expires at Sun 29 Aug 2021 09:14:53 AM PDT
Is this correct? (y/N) y

GnuPG needs to construct a user ID to identify your key.

Real name: Dante Alighieri
Email address: dante564@gmail.com
Comment: Personal key
You selected this USER-ID:
    " Dante Alighieri (Personal key) <dante564@gmail.com>"

Change (N)ame, (C)omment, (E)mail or (O)kay/(Q)uit? O
We need to generate a lot of random bytes. It is a good idea to perform
some other action (type on the keyboard, move the mouse, utilize the
disks) during the prime generation; this gives the random number
generator a better chance to gain enough entropy.
We need to generate a lot of random bytes. It is a good idea to perform
some other action (type on the keyboard, move the mouse, utilize the
disks) during the prime generation; this gives the random number
generator a better chance to gain enough entropy.
gpg: key 4B5U18A28E72GG8D marked as ultimately trusted
gpg: directory '/home/dante/.gnupg/openpgp-revocs.d' created
gpg: revocation certificate stored as '/home/dante/.gnupg/openpgp-revocs.d/HF89G265EA9CDC2FA20D82414D5B18A38E72FF6Z.rev'
public and secret key created and signed.

pub   rsa4096 2021-08-22 [SC] [expires: 2021-08-29]
      BH99F864EA9EBE2FH20A74214B5U18A28E72GG8D
uid   Dante Alighieri (Personal key) <dante564@gmail.com>
sub   rsa4096 2021-08-22 [E] [expires: 2021-08-29]
```

During this process you will be prompted for a passphrase. Choose one that has at least 6 words and includes at least one number.

### Encrypting and Decrypting Files Using GPG ###
You can now encrypt a file using your public key. Here is an example using the key that I just created,
```
echo "this is a secret note!" > secret.txt
gpg --encrypt --recipient dante564@gmail.com --output secret.txt.gpg secret.txt
```
This creates an encrypted version of my text file `secret.txt` that is named `secret.txt.gpg`. The private key for `dante564@gmail.com` and its associated passphrase are needed to decrypt this file. If you enter
```
gpg --decrypt secret.txt.gpg
```
you will be prompted for the passphrase and the file will be decrypted to stdout (displayed on screen) if you want to save it to a file instead use redirection,
```
gpg -d secret.txt.gpg > secret2.txt
```
The emacs package [EasyPG Assistant](https://www.gnu.org/software/emacs/manual/html_mono/epa.html) provides automatic encryption and decryption of *.gpg files that are opened in emacs and will be covered later in the course. For more details on GPG commands there is a nice cheat sheet available [here](https://blog.programster.org/gpg-cheatsheet)

### Sharing Public Keys ###
For someone to send you an encrypted file that ONLY YOU can read using your private key they must use your public key to encrypt the file. To share your public key
with them you first need to export the key into a text (ascii) format and then send them the text file that contains the public key. To export a key in text format
(rather than binary use the `--armour` option. For example, to export the public key for brannala@gmail.com I would use,
```
gpg --export --armour "dante564@gmail.com" > dante564@gmail.com.pub.asc
```
To share this public key I would send it to my colleague and they would import it into their gpg key database using,
```
gpg --import dante564@gmail.com.pub.asc
```
The key is now available for them to use in creating an encrypted file that only I can decrypt using the secret key for dante564@gmail.com.

## SSH Using Keys ##
Asymmetrical encryption is also used for creating keys to allow ssh logins without passwords. This is generally considered safer than password logins because
brute force password guessing algorithms could compromise a weak password while a private key must be physically present on a local machine to allow login via
SSH using keys. Also, a secure private key will also have a passphrase that must be entered to use it so even if an intruder gains access to your private key
they must still crack the passphrase to use it for logging in to a remote machine. To use ssh with keys you must first create a pair of keys and then copy the
public key to the remote machine. It is best to have the private key on as few machines as possible (ideally one laptop). 

### Creating an SSH key pair ###
To create a new key pair use the command (with your email address, obviously):
```
ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
```
You will first be prompted for the path/name of a file in which to save the key. The file name is arbitrary but the path you specify should lead to the subdirectory ```.ssh``` in your home directory. It is okay to use the default if this is the first key pair that you have generated. Next you are prompted for a passphrase. Don't leave the passphrase blank (default) or your private key can then be used without a passphrase, which is unsafe. Here is an example session:
```
dante@home:~$ ssh-keygen -t rsa -b 4096 -C "dante564@gmail.com"
Generating public/private rsa key pair.
Enter file in which to save the key (/home/dante/.ssh/id_rsa): /home/dante/.ssh/test
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /home/dante/.ssh/test
Your public key has been saved in /home/dante/.ssh/test.pub
The key fingerprint is:
SHA256:Xi+GXMlZvFRJUGACXyraMiQkC6pzuxhggMrsHny8qDA dante564@gmail.com
The key's randomart image is:
+---[RSA 4096]----+
| o ....+.        |
|o . ooo..        |
| .   oo..+       |
|+..   .+* .      |
|*o    *+So       |
|G..    o         |
|E+ o             |
|&o* .            |
|*&o+             |
+----[SHA256]-----+
```
You should now see two new keys in the subdirectory. After executing the example above the result is:
```
ls ~/.ssh
config known_hosts  test  test.pub
```
The file named ```test``` contains the private key and ```test.pub``` contains the public key. Keep the private key private, obviously. Don't put it on the internet or share it with anyone. Ideally keep it on just one computer and create a backup on a removable disk. We are now ready to make use of these keys.

### Using SSH keys with the git command line tools ###
Here I quickly summarize how to use ssh keys with github and bitbucket. Let's start with github. 

#### Setting up SSH keys with Github ####
Here is a step-by-step guide to setting up SSH keys for use with Github:

##### Upload your public key to github #####
Upload your public key to your github account. To do this log in to github through the web interface and from the drop down menu at the upper right of your home page choose ```Settings```. The settings page will show a list of ```Account settings``` on the left. Choose the button labelled ```SSH and GPG keys```. Click on the button ```New SSH Key``` and enter the name of your key in the ```Title``` box. Then open up your public key in a text editor and copy the contents into the clipboard, then paste it into the ```Key``` box. Save the key.

##### Initialize the key for each login session on your local machine #####
To initialize the key for use by ssh-agent use the following command where ```mykey``` is the name of the key you created:
```
ssh-add ~/.ssh/mykey
```
You will be prompted for the passphrase. To avoid typing this at each login, create a bash script named gitssh with the following contents:
```
#!/bin/bash
ssh-add ~/.ssh/mykey
```
make this read-write-executable by owner and move it to your local bin directory (hopefully ~/bin is in your shell PATH variable): 
```
chmod 700 gitssh
mv gitssh ~/bin
```
Then if you type ```gitssh``` you will be prompted for the passphrase once and can then use
git commands such as push without having to authenticate.

##### Change origin for existing HTTPS repositories to SSH #####
For any existing repositories (previously cloned as HTTPS) change the remote site address from HTTPS to SSH. For example, if the original HTTPS address was:
```
https://github.com/dantesinferno/mpestw.git
```
the new SSH address will be:
```
git@github.com:dantesinferno/mpestw.git
```
To make the above conversion change to the git local repository directory and use the ```remote set-url``` git options:
```
cd ~/repos/mpestw
git remote set-url origin git@github.com:dantesinferno/mpestw.git
```
You can verify that the remote has changed using
```
git remote -v
```
When creating local repositories via cloning in future use the SSH address.

#### Setting up SSH keys with Bitbucket ####
Setting up ssh keys in Bitbucket is very similar. Log in to the web interface and now look for an account link on the lower left of the home page which brings up a menu with the selection ```Personal setting``` and then you will see a link for SSH keys. Clicking on this link reveals an Add key button, etc. Once you have added the public key to your Bitbucket account you will again need to modify the origin for any existing Bitbucket repository clones on your local machine. The ssh address will be of the form:
```
git@bitbucket.org:dantesinferno/myrepository.git
```

### Logging into a remote machine using ssh keys ###
To log in to a remote machine using ssh keys you first need to copy the public key to each server. For example if my username on server1 (ip address 128.128.1.4) is ```skinny``` and my key is named ```test``` I can copy the key using
```
ssh-copy-id -i ~/.ssh/test skinny@128.128.1.4
```
and then try logging in using the key
```
ssh -i ~/.ssh/test skinny@128.128.1.4
```
which should allow you to log in without using your password. To make your life even easier create a file named  ```~\.ssh\config``` that contains an entry for each machine that you will log in to via ssh of the following form (with a blank line between each entry in the file):
```
host server1
Hostname 128.128.1.4
User skinny
IdentityFile ~/.ssh/test
```
Now to log in to server1 you just type ```ssh server1```.
