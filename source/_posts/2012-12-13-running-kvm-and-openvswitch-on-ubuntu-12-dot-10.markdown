---
layout: post
title: "Running KVM and Openvswitch on Ubuntu 12.10"
date: 2012-12-13 22:40
comments: true
categories: [virtualization, kvm, ubuntu, openvswitch] 
---


I've got an aging VMWare ESXi 4.0 server that needs to be replaced with something a little more modern and flexing.   Obviously at home I don't need all the cool features that licensed VMWare comes with,  but I do want more than just the basic free version.


After a few weeks of installing and testing alternatives  ( I'd really love to run openstack,  but it's just not worth it at home for a single box ) I've settled on Ubuntu 12.10 server running KVM and Openvswitch.


After installing Ubuntu 12.10 I did the following to get KVM up and running...   I cribbed this mostly from [blog.allanglesit.com](http://blog.allanglesit.com/2012/10/linux-kvm-ubuntu-12-10-with-openvswitch/).


<!--more-->

**Install updates and Pre-requisites**


First of all, make sure Ubuntu is fully up to date:
*sudo to root as pretty much every command here needs to be run as root*

{% codeblock lang:bash %}
sudo bash
apt-get update
apt-get upgrade
{% endcodeblock %}

Now we can go ahead and install the necessary packages:


{% codeblock lang:bash %}
apt-get -y install aptitude apt-show-versions ntp ntpdate vim kvm \
 libvirt-bin vlan virtinst virt-manager virt-viewer openssh-server \
 iperf pv openvswitch-controller openvswitch-brcompat \
 openvswitch-switch nfs-common 
{% endcodeblock %}

**Kill off the default libvirt bridge and nuke ebtables**


We want to delete the default libvirt interface and we don't need ebtables so we'll get rid of that.

{% codeblock lang:bash %}
virsh net-destroy default
virsh net-autostart --disable default
service libvirt-bin stop
service qemu-kvm stop
aptitude purge -y ebtables
service openvswitch-switch restart
service openvswitch-controller restart
{% endcodeblock %}

**Configure network interfaces**

We'll be using just a single interface which will be used for both the bridge and the host itself.    We will also be bridging that network into the the vswitch and then configuring an interface for the host OS.    The network configuration will look something like this:


{% codeblock /etc/network/interfaces %}
# The loopback network interface
auto lo
iface lo inet loopback

# The primary network interface - bridge!
auto eth0
iface eth0 inet manual
up ifconfig $IFACE 0.0.0.0 up
down ifconfig $IFACE down
    
# The host OS network interface
# DNS settings here,  12.10 resets resolv.conf on reboot.
auto ovsbr0p1
iface ovsbr0p1 inet static
address 192.168.50.10
netmask 255.255.255.0
gateway 192.168.50.1
dns-nameservers 192.168.50.1
dns-search example.com
{% endcodeblock %}

**Configure the openvswitch network**


Now we need to configure the network on the openvswitch.     We need to define the bridge, connect it to the uplink interface and create a port for the host OS.


*note: if you're doing this via SSH it will probably break your session*

{% codeblock lang:bash %}
ovs-vsctl add-br ovsbr0
ovs-vsctl add-port ovsbr0 eth0
ovs-vsctl add-port ovsbr0 ovsbr0p1 -- set interface ovsbr0p1 type=internal
reboot
{% endcodeblock %}

**Modify network service sleep times**


That took forever to boot.    We can fix that by modifying sleeps in /etc/init/failsafe.conf and reboot again to make sure it helped.


Change :

{% codeblock %}
$PLYMOUTH message --text="Waiting for network configuration..." || :
sleep 40
$PLYMOUTH message --text="Waiting up to 60 more seconds for network configuration..." || :
sleep 59
$PLYMOUTH message --text="Booting system without full network configuration..." || :
{% endcodeblock %}

To :

{% codeblock %}
$PLYMOUTH message --text="Waiting for network configuration..." || :
sleep 1
$PLYMOUTH message --text="Waiting up to 60 more seconds for network configuration..." || :
sleep 1
$PLYMOUTH message --text="Booting system without full network configuration..." || :
{% endcodeblock %}


**LVM configure**


We're going to also use LVM to for the KVM virtual machines to use as storage.    I have a pair of 500g disks in a software raid1 which I'll use for this.

{% codeblock lang:bash %}
pvcreate /dev/md0
vgcreate data-disk /dev/md0
lvcreate -L 10G -n ISO data-disk
mkfs.ext4 /dev/data-disk/ISO
mkdir -p /data-disk/ISO
echo "/dev/data-disk/ISO /data-disk/ISO	defaults	ext4	0 0" >> /etc/fstab
mount -a
{% endcodeblock %}

**Create VM**


Now we can go ahead and create our first VM.   I've already downloaded the Ubuntu ISO to /data-disk/ISO 


*note: virt-install does not support setting a virtualport type of openvswitch yet .. so we have to trick it*

{% codeblock lang:bash %}
lvcreate -L 8G -n VM-UbuntuTest data-disk
virt-install --name UbuntuTest --hvm --noautoconsole --ram 1024 \
--disk path=/dev/data-disk/VM-UbuntuTest --nonetworks --vnc \
--os-type=linux --os-variant=ubuntuquantal \
--cdrom /data-disk/ISO/ubuntu-12.10-server-amd64.iso
{% endcodeblock %}

set up the networking by editing the VM's XML and adding a network interface stanza just before the \</devices\>.

{% codeblock lang:bash %}
virsh edit UbuntuTest
{% endcodeblock %} 

{% codeblock lang:xml %}
<interface type='bridge'>
 <source bridge='ovsbr0'/>
 <virtualport type='openvswitch' />
 <model type='virtio'/>
</interface>
{% endcodeblock %} 
   
The VM will need to be reset to pick up the network change,  however that will cause it to drop the ISO mount.  We can either continue through with the OS install without networking or reset the VM and then re-attach the ISO as a CD.    I connected to it from my desktop using the VirtualMachineManager GUI to do that but you could use virsh commands if you want to stick to CLI.

