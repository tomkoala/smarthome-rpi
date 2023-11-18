# Headless RPI setup on Mac

## 1. Check SD card 

Insert the SD card in the reader (e.g. USB)

Retrieve the disk identifier with `diskutil`

```
diskutil list
```

Use `f3` to test the card before putting it into service. On macOS, it can be installed via `brew install f3`; and then using it looks like this:

```
f3write /Volumes/sdcard/ && f3read /Volumes/sdcard/
```
where `sdcard` is the named volume of the SD card

## 2. Copy the image

Unmount the volume before cloning the image

```
diskutil unmountDisk /dev/disk#
```

where `#` is the identifier of the SD card

Burn the image to the micro SD card

```
sudo dd bs=1m if=path_of_your_image.img of=/dev/rdisk# conv=sync
```

> :memo: **Notes:**
> - /dev/rdisk nodes are much faster than /dev/disk
> - Control-T to learn the copy status

## 3. Before first boot

Enable SSH by placing a file named `ssh` (without any extension) onto the boot partition of the SD card.
Set up the Wifi by copying `wpa_supplicant.conf` in the same partition then edit the network psk (*password*)

## 4. First SSH login

Open a new SSH session with user `pi` after a fresh install :

```
ssh pi@raspberrypi.local
```

At this stage a default password shall be used : `raspberry`

The first time you open a SSH session, you may get the following disclamer :

```
The authenticity of host 'raspberrypi.local (192.168.X.X)' can't be established.
ECDSA key fingerprint is SHA256:pERC7+nTzU/qRy0ZvzbYbqpI4yUYZUDpNM4ONcPFFb0.
Are you sure you want to continue connecting (yes/no/[fingerprint])?
```
Type `yes` followed by `Enter`.

## 5. Configure Pi

To change hostname (e.g. `domopi`) and password as well as configure locales launch the following script :
```
sudo raspi-config
```

## 6. Generate SSH keys

The following steps allow to configure SSH key-based authentication on Raspberry Pi.

The overall process is as follows:
- Generate private and public keys on the Host / client machine
- Install the public key on the pi
- Disable the SSH password-based authentication
- Configure SSH on the client machine for easy access (optional)

The simplest way to generate a key pair is to run `ssh-keygen` without arguments. In this case, it will prompt for the file in which to store keys. However we will use the following command to select the RSA algorithm : 

```
ssh-keygen -t rsa -b 4096
```

Simply enter the passphrase or press `enter` twice to apply an empty passphrase (**not recommended**)

```
Generating public/private rsa key pair.
Enter file in which to save the key (/Users/<username>/.ssh/id_rsa): 
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /Users/<username>/.ssh/id_rsa.
Your public key has been saved in /Users/<username>/.ssh/id_rsa.pub.
The key fingerprint is:
SHA256:Cu1TXpeaSVXaj2I1dwp3/whUAAbykATCLZYde63fbSo tom@imac24.home
The key's randomart image is:
+---[RSA 4096]----+
| ..+o++...o...o  |
|  =.o..= .   =   |
| . .. . o   = = +|
|     o .   o = Bo|
|    . o S o * o o|
|     o = + B o ..|
|      + o = o . .|
|       .E  o     |
|         ..      |
+----[SHA256]-----+
```

Your private key will end up in a file like `/Users/<username>/.ssh/id_rsa` and the corresponding public key in `/User/<username>/.ssh/id_rsa.pub`. 

> :warning: **Warning** Never share your private key with anyone!

### Register keys 

Add your private key to the SSH Agent using :

```
ssh-add -K ~/.ssh/id_rsa
```

The SSH Agent is supposed to automatically load the keys and store the corresponding passphrases in your Mac OS keychain.

Finally you can check if the agent successfully stored the key by listing all identities:

```
ssh-add -l
```

## 7. Add the SSH public key on Pi

Now it’s time to configure the public key on the remote Pi machine to allow key-based authentication.
Copy the content of the public key `id_rsa.pub` and paste that into a file called `authorized_keys` located in the `.ssh` folder

To configure the **Pi** to accept public key authentication, uncomment the following lines in the `sshd_config` file

```
PubkeyAuthentication yes
AuthorizedKeysFile      .ssh/authorized_keys .ssh/authorized_keys2
```

### To disable password login (**optional**)

> :warning: 
> Check that the key-based authentication is already working before modifying these settings

Find the following lines and change them as follows,

```
PermitRootLogin no
PasswordAuthentication no
ChallengeResponseAuthentication no
UsePAM no
```

Finally restart the SSH service to load the latest settings :

```
sudo systemctl reload sshd
```

### Test the key-based authentication on client side

```
ssh-keygen -R domopi.local
ssh pi@domopi.local
```
