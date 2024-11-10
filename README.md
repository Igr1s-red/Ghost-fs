# casper-fs 👻
Casper-fs is a custom Linux Kernel Module generator to work with resources to protect or hide a custom list of files.
Each LKM has resources to protect or hide files following a custom list in the YAML rule file. Yes, not even the root has permission to see the files or make actions like edit and remove. The files only can be caught, edited, and deleted if the user sends a proper key to the custom device to liberate the action in the file system. 

## Overview

The first point, the user sends input to the Casper-FS the YAML file with rules to generate a custom LKM(Linux kernel module), each generated module works in the file system to protect and hide secret files (Not even the root has permission to see the file, only can see with a proper sending key to the custom device). This program has two principal functions: turning private files hidden. The second function is to protect confidential files to prevent reading, writing and removal.
  

The motivation: An attacker can read every file in your machine in a bad situation(if he got root). But if you have a Casper-fs custom module, the attacker will not find the hidden kernel module that has functions to protect your private data files such as logs and password vaults.

My beginning purpose at this project is to protect my server, which is to protect my friends' machines.
When I talk to friends, I say peoples that don't know how to write low-level code. Using the Casper-fs, you can 
generate your custom kernel module to protect your secret files. The low-level programmer can write new templates for modules etc.

The first step, understand before the run.
--

Verify if the kernel version is 3.x, 4.x, or 5.x:
```
$ uname -r
```

Clone the repository
```
$ git clone https://github.com/CoolerVoid/casper-fs
```

Enter the folder and install python3 modules:
```
$ cd casper-fs/module_generator
$ sudo python3 -m pip install -r requirements.txt
```

Edit your file rules in directory module_generator/rules/fs-rules.yaml, the python scripts, use that file to generate a new casper-fs custom module.

```
$ cat module_generator/rules/fs-rules.yaml
binary_name: casperfs
module_name: Casperfs
unhide_module_key: AbraKadabra
hide_module_key: Shazam
fake_device_name: usb15
unhide-hide-file-key: Alakazam
unprotect-protect-file-key: Sesame 
fs-rules: 
- hidden: 
   1: secret.txt
   2: my_vault.db
- protect:
   1: backup_httpd.log
```
The array is hidden and array protected. You can insert a lot of the elements of another file on context, for example:
```
- protect:
   1: backup_httpd.log
   2: secret_img.iso
   3: secret_file.img
   4: secret_file2.img
   5: secret_file3.img
```
If you want to study the static code to generate, look at the directory "templates" content.

The second step, generate your module.
--

If you want to generate a kernel module following your YAML file of rules, follow that command:

```
$ python3 casper-fs-gen.py --rules rules/fs-rules.yaml
```
This action can generate a generic module with the fs-rules.yaml.


The third step, install your module.
--

If you use Fedora Linux, install kernel packages for the developer:
```
# dnf update
# dnf install kernel-headers.x86_64 kernel-modules.x86_64 kernel.x86_64 kernel-devel kmod
```
On Ubuntu Linux:
```
apt install linux-headers-generic gcc make
```
To test module:
```
# cd output; make clean; make
# insmod casperfs.ko
```


The fourth step runs your custom module.
--

* The password to turn casper-fs module visible for lsmod is the key "Shazam".
* The password to turn the casper-fs invisible is "AbraKadabra".
* The password to turn the secret files in hidden is "Alakazam", the same to turn to unhidden.
* The password to protect files or unprotect is "Sesame".

You need to send the password for your fake device, "usb15" for example, to test hidden and unhidden resources on the file system:
```
$ touch secret.txt
$ ls
-- no results--
$ echo "Alakazam" > /dev/usb15
$ ls
secret.txt
$ echo "Alakazam" > /dev/usb15
$ ls
-- no results--
```
So this is an example of trying to remove a protected file by Casper-fs:
```
# ls
test.txt  log.txt backup_httpd.log
# rm backup_httpd.log
# ls 
test.txt  log.txt backup_httpd.log
# echo "Sesame" > /dev/usb15            // to remove protection
# rm backup_httpd.log
# ls
test.txt  log.txt
# echo "Sesame" > /dev/usb15    // to active protection again
```

* Note
You need to turn casperfs visible at the "lsmod" command. Need this action before removing module

```
# rmmod casperfs
rmmod: ERROR: ../libkmod/libkmod-module.c:799 kmod_module_remove_module() could not remove 'casperfs': No such file or directory
rmmod: ERROR: could not remove module casperfs: No such file or directory
# lsmod | grep casper
# echo "Shazam" > /dev/usb15
# lsmod | grep casper
casperfs
# rmmod casperfs
```

# Random notes

Tested on ubuntu 16 and fedora 29 at kernels "3.x","4.x" and "5.x".

# Persistence and hide command of history
Follow this tip for more information:
https://github.com/CoolerVoid/casper-fs/blob/main/module_generator/scripts/README.md


## Point of attention
This tool aims to use in the hardening system context. Pay attention if you have proper authorization before using that. I do not have responsibility for your actions. You can use a hammer to construct a house or destroy it, choose the law path, don't be a bad guy, remember.


# References

*Wikipedia Netfilter* 
https://en.wikipedia.org/wiki/Netfilter

*Linux Device Drivers* 
http://lwn.net/Kernel/LDD3/

*M0nad's Diamorphine* 
https://github.com/m0nad/Diamorphine/
