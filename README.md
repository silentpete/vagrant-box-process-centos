# CENTOS 7 1804 VIRTUAL BOX PROCESS

This is an at home effort to get more familiar with creating Vagrant Boxes.

In this write up, you should be able to follow along and end up with an CentOS 7 base development environment up and running.

## STEPS

This process was completed from a Windows 10 environment. To start, download and install the required software.

- CentOS (tested with: 1804 - https://www.centos.org/)
- Vagrant (tested with: 2.1.1 - https://www.vagrantup.com/)
- VirtualBox (tested with: 5.2.12 - https://www.virtualbox.org/)
- VirtualBox Extention Pack (tested with: 5.2.12)
- VirtualBox Guest Additions (http://download.virtualbox.org/virtualbox/5.2.12/VBoxGuestAdditions_5.2.12.iso)

The Handheld Manual Process:

1. Start Oracle VirtualBox
1. Create a new virtual machine
    1. New
    1. Name: centos7 (may auto detect OS Type and Version), click next
    1. Adjust memory: 2GB, click next
    1. Create a virtual hard disk now, click create
    1. VMDK, click next
    1. Dynamic, click next
    1. Change disk space as desired: tested with 40GB, click create
1. Adjust setting before starting machine
    1. turn off audio
    1. turn off USB
    1. Network / Adapter 1 should be NAT
    1. Adapter 1 / Advanced / Port forwarding should have a SSH TCP rule with host 2222 and guest 22 ports assigned.
1. start virtual machine
1. Should ask to connect ISO (could connect before starting)
1. Install CentOS
    1. Choose language
    1. Click installation destination and hit done to accept defaults
    > Unless you want to adjust the partition layout... remember that if you fill '/' you have a sad system. Also, remember you can mount in space on the host.
    1. Begin Installation
    1. Set root password to vagrant
    1. Create user vagrant with password vagrant and make user an Admin
    1. reboot when finished
1. login
1. set passwordless sudo
    1. sudo visudo
    ```none
    # add vagrant user
    vagrant ALL=(ALL) NOPASSWD:ALL
    ```
    > May have to log out and back in
1. By default, CentOS ships with network ONBOOT=no, turn on the nic
    > While you're in the file, might as well clean it up for this environment, remove IPV6, UUID.

    ```none
    sudo vi /etc/sysconfig/network-scripts/ifcfg-e<tab>
    ```
    1. Set ONBOOT to yes
    1. save
    1. Restart the network service
        ```none
        sudo systemctl restart network
        ```
1. Run updates
    ```none
    sudo yum -y update
    ```
1. Add desired applications
    ```none
    sudo yum -y install bzip2 gcc kernel-headers kernel-devel wget vim

    sudo reboot
    ```
    > Note: while in the virtual machine, prior to saving it as a box, this would be a good time to add in other packages you may want such as: docker-ce, htop, httpd-tools, git, maven, ruby, ansible
1. Install the HashiCorp public key (authorized_keys)
    ```none
    cd ~

    mkdir -p /home/vagrant/.ssh

    sudo chmod 0700 /home/vagrant/.ssh

    wget --no-check-certificate https://raw.githubusercontent.com/hashicorp/vagrant/master/keys/vagrant.pub -O /home/vagrant/.ssh/authorized_keys

    sudo chmod 0600 /home/vagrant/.ssh/authorized_keys

    sudo chown -R vagrant /home/vagrant/
    ```
1. Restart sshd
    ```none
    sudo systemctl restart sshd
    ```
1. Install VirtualBox Guest Additions
    1. mount the iso
        1. Add the disk to the virtual box
            1. In VirtualBox Manager, click setting for the centos virtual machine
            1. Click storage
            1. click Empty under the IDE controller
            1. on the right hand side, next to Optical Drive, click the image and navigate to the VirtaulBox Guest Additions ISO
            1. Click ok
        1. Back in the VM, create the directory where we will mount the cdrom
            ```none
            sudo mkdir -p /mnt/cdrom/

            sudo mount /dev/cdrom /mnt/cdrom
            ```
            > expect: 'mount: /dev/<vol> is write-protected, mounting read-only'

    1. Run the additions
        ```none
        sudo sh /mnt/cdrom/VBoxLinuxAdditions.run
        ```
        > expect: 'Could not find the X.Org or XFree86 Window System, skipping.' Can test that the serice is running with "VBoxService --help" or VBoxControl --help"
1. Adjust SELinux if desired
    1. Set to Permissive
        ```none
        sudo setenforce Permissive
        ```
1. Clean up before packaging
    1. remove any unnessesary packages
    1. remove yum cache
        ```none
        sudo yum clean all

        sudo rm -rf /var/cache/yum
        ```
    1. Unmount drive
        ```none
        sudo umount /dev/cdrom
        ```
    1. Remove ISO from VM in VirtualBox Manager
    1. Final Step before packing is to fill the hard drive with zeros, this helps with compression, also clear the history.
        ```none
        sudo dd if=/dev/zero of=/EMPTY bs=1M

        sudo rm -f /EMPTY

        sudo cat /dev/null > ~/.bash_history && history -c
        ```
1. From your host environment, package the vagrant box
   1. Open a GIT Bash or CMD
   1. navigate to where you want to store you vagrant box files (ie. /vagrant_boxes)
   1. run the following command to package the vagrant box
    ```none
    vagrant package --base <name of the virtual box> --output <name of file>
    ```
    > you should now have a package.box file in your directory or whatever you named it

## TESTING

To test the box, pretend you are a new user of Vagrant (be daring and delete your centos7 VM first) and give it a shot:

```none
vagrant box add <name> /path/to/the/package.box

vagrant init <name>

vagrant up

vagrant ssh

ping google.com
```

## MY VAGRANT NOTES

https://www.vagrantup.com/docs/virtualbox/boxes.html

Vagrant Recommends using Packer to create reproducible builds for your base boxes.
Vagrant Recommends using Atlas to automate the builds

Vagrant Expected Defaults (https://www.vagrantup.com/docs/boxes/base.html)

- "vagrant" user with vagrant password
- Password-less Sudo, add "vagrant ALL=(ALL) NOPASSWD: ALL" (with no ") to the #visudo
- vagrant doesn't use a pty or tty, so if you want one you have to configure it into the Vagrantfile
- to keep ssh speedy, set the UseDNS configuration to no in the SSH server configuration (This avoids a reverse DNS lookup on the connecting SSH client which can take many seconds.)

If creating a box for Virtual Box, make sure it has:

- VirtualBox guest additions

Disk Space
They recommend making a volume that can be dynamically grown or mounting in volume space

Memory
Assign what is needed

Disable any non essencial hardware

### Troubleshooting

Error: `default: Warning: Authentication failure. Retrying…`

Resolution: make sure "Install the HashiCorp public key" step were correctly performed

## REFERENCES

- https://www.engineyard.com/blog/building-a-vagrant-box
- Vagrant Up and Running (book)
- vagrantup.com
- virtualbox.org
