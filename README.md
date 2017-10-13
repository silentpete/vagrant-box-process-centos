Vagrant Recommends using Packer to create reproducible builds for your base boxes.
Vagrant Recommends using Atlas to automate the builds

Vagrant Expected Defaults (https://www.vagrantup.com/docs/boxes/base.html)
- "vagrant" user with vagrant password
- Password-less Sudo, add "vagrant ALL=(ALL) NOPASSWD: ALL" (with no ") to the #visudo
- vagrant doesn't use a pty or tty, so if you want one you have to configure it into the vagrantfile
- to keep ssh speedy, set the UseDNS configuration to no in the SSH server configuration (This avoids a reverse DNS lookup on the connecting SSH client which can take many seconds.)

if creating a box for Virtual Box, make sure it has:
- VirtualBox guest additions

Disk Space
They recommend making a volume that can be dynamically grown or mounting in volume space

Memory
Assign what is needed

Disable any non essencial hardware

# VIRTUAL BOX PROCESS (https://www.vagrantup.com/docs/virtualbox/boxes.html)
Box File (tar, tar.gz, zip)
- Within the archive, Vagrant does expect a single file: metadata.json. This is a JSON file

Minimum contents of metadata.json
```
{
  "provider": "virtualbox"
}
```

Box Catalog Metadata

Full example
```
{
  "name": "hashicorp/precise64",
  "description": "This box contains Ubuntu 12.04 LTS 64-bit.",
  "versions": [
    {
      "version": "0.1.0",
      "providers": [
        {
          "name": "virtualbox",
          "url": "http://somewhere.com/precise64_010_virtualbox.box",
          "checksum_type": "sha1",
          "checksum": "foo"
        }
      ]
    }
  ]
}
```
url key in the JSON can also be a file path

Box Information

## VirtualBox Requirements
- The first network interface (adapter 1) must be a NAT adapter. Vagrant uses this to connect the first time.

- The MAC address of the first network interface (the NAT adapter) should be noted, since you will need to put it in a Vagrantfile later as the value for config.vm.base_mac. To get this value, use the VirtualBox GUI.

- VirtualBox Guest Additions for Linux
## STEPS
Download Vagrant (I tested with 2.0.0)
Download VirtualBox (I tested with 5.1.28)
Download VirtualBox Extention Pack (5.1.28-117968)
Download CentOS (I tested with 1708)

1. Start Oracle VirtualBox
1. Create a new virtual machine
1. added a 30 GB hard drive VDI, and 4 GB memory
   1. need to figure out partitioning, can't have everything on /
1. turn off USB
1. turn off floppy drive
1. turn off audio
1. start virtual machined
1. login
1. set passwordless sudo
   1. sudo visudo
```
# add vagrant user
vagrant ALL=(ALL) NOPASSWD:ALL
```
   1. may have to log out and back in
1. turn on the nic
   1. sudo -i
   1. vi /etc/sysconfig/network-scripts/ifcfg-e<tab>
   1. turn onboot to yes
1. run updates


# TESTING
To test the box, pretend you are a new user of Vagrant and give it a shot:
```
$ vagrant box add my-box /path/to/the/new.box
...
$ vagrant init my-box
...
$ vagrant up
...
```
