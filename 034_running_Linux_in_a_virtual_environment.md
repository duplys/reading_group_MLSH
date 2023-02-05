# Installing VirtualBox an Ubuntu 20.04 with UEFI Secure Boot enabled
I'm installing VirtualBox on an Ubuntu 20.04 machine:
```shell
dup2rt@RNG-C-000VG:~$ lsb_release -a
No LSB modules are available.
Distributor ID:	Ubuntu
Description:	Ubuntu 20.04.5 LTS
Release:	20.04
Codename:	focal
```

To install VirtualBox, including guest extensions, do:
```shell
dup2rt@RNG-C-000VG:~$ sudo apt update
dup2rt@RNG-C-000VG:~$ sudo apt install virtualbox virtualbox-qt virtualbox-dkms virtualbox-guest-x11 -y
dup2rt@RNG-C-000VG:~$ sudo adduser $USER vboxusers 
```

Because my machine has UEFI Secure Boot enabled, the VirtualBox `vboxdrv` kernel module will not be loaded because it is not signed. You can find details how to fix it [in this blog post]( https://gist.github.com/jeffersfp/9ee1fe859f4e480267e23a58b4b36c93) or in this [StackExchange post](https://superuser.com/questions/1438279/how-to-sign-a-kernel-module-ubuntu-18-04).

First, you need to generate your private key that you want to use for signing the vbox kernel module:
```shell
$ openssl req -new -x509 -newkey rsa:2048 -keyout MOK.priv -outform DER -out MOK.der -nodes -days 36500 -subj "/CN=YOUR_NAME/"
```

For example, you can do:
```shell
$ openssl req -new -x509 -newkey rsa:2048 -keyout MOK.priv -outform DER -out MOK.der -nodes -days 36500 -subj "/CN=PD/"
```

Now use `mokutil`, a tool to import or delete the machine owner keys (MOK), to import the public key, and then enroll it when the machine is rebooted. The password in this step is a temporary use password you'll only need to remember for a few minutes.
```shell
$ sudo mokutil --import MOK.der
input password:
input password again:
```

Next, save the following code for actually signing the `vboxdrv` kernel module in a shell script, in the same directory where the newly generated key files `MOK.der` and `MOK.priv` are located. In my case, I called the shell script `sign_vbox_modules.sh`:

```shell
#!/bin/bash

for modfile in $(dirname $(modinfo -n vboxdrv))/*.ko; do
  echo "Signing $modfile"
  /usr/src/linux-headers-$(uname -r)/scripts/sign-file sha256 ./MOK.priv ./MOK.der "$modfile"
done
```

Now you need to reboot your computer. As described in this [StackExchange post](https://superuser.com/questions/1438279/how-to-sign-a-kernel-module-ubuntu-18-04):

	When the bootloader starts, you should see a screen asking you whether you want to enter the MOK manager EFI utility. Note that any external external keyboards won't work in this step. Select Enroll MOK in the first menu, then continue, and then select Yes to enroll the keys, and re-enter the password established in step 2. Then select OK to continue the system boot.


After enrolling the keys and booting your computer again, change into the directory where your `MOK.priv`, `MOK.der`, and `sign_vbox_modules.sh` files are stored and run:

```shell
$ chmod 700 sign_vbox_modules.sh
$ sudo ./sign_vbox_modules.sh
Signing /lib/modules/5.4.0-137-generic/updates/dkms/vboxdrv.ko
Signing /lib/modules/5.4.0-137-generic/updates/dkms/vboxnetadp.ko
Signing /lib/modules/5.4.0-137-generic/updates/dkms/vboxnetflt.ko
```

Now, the following command should execute without errors:
```shell
$ sudo modprobe vboxdrv
```

If this is the case, you can start Virtualbox and fire up your virtual machine!

If you install centOS 7 and there is no Internet connection after you boot the centOS 7 VM, you need to do what is described for the headless in [this post](https://unix.stackexchange.com/questions/78295/centos-no-network-interface-after-installation-in-virtualbox).
