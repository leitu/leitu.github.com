##Install CoreOS on HV
We are using ipxe boot as installation method, which is easy for future usage.

XenServer: 6.2
CoreOS: v557.02, stable


#Configure your ipxe server
Insprition with @kelseyhightower, his present in Gophercon 2014, use his ipxeserver simple code to running a ipxe server 1, you also can visit his project 2.

You need to specify your own key location before running complier, This light web service host the boot file

``
  go build ipxeserver.go
  ./ipxeserver
``

#Get all your needs image to local
Here we are using xen as a example

We download all we needs files to local

``bash
  wget http://stable.release.core-os.net/amd64-usr/current/coreos_production_xen_image.bin.bz2.sig
  wget http://stable.release.core-os.net/amd64-usr/current/coreos_production_xen_image.vhd.bz2.sig
  wget http://stable.release.core-os.net/amd64-usr/current/coreos_production_pxe.vmlinuz
  wget http://stable.release.core-os.net/amd64-usr/current/coreos_production_pxe_image.cpio.gz
``

Assume we are having a web server to host those, or we can use ipxe server to host this.


``bash
 wget http://boot.ipxe.org/ipxe.iso
``

Create a new VM in XenCenter use "other install media", and set 4G/40G as memory and hard disk, inject ipxe.iso as well

once the server boot up, send "Ctrl+B" to the console.
type
IPXE> dhcp
IPXE> chain http://10.18.2.179:9000/boot

This is defined in ipxeserver

It will automatic install CoreOS in RAM.

Since you inject your ssh key in CoreOS.

ssh core@10.18.2.225 (suppose you get this DHCP)

You need cloud-init.yml ready.

``
 wget http://10.18.2.123/cloud-init.yml
``

on 10.18.2.225

``
 sudo -s
 coreos-install -b http://10.18.2.179 -C stable -d /dev/xvda -c cloud-init.yml -o xen
``

Then CoreOS is up and running on xenserver, this OEM version is xentools installed, you can find it's optimized, but it can't fetch IP address in XenCenter, looking on this

If you are choosing vmware or vagrant, just change xen to other.

1. https://github.com/gophercon/2014-talks/tree/master/kelseyhightower/ipxeserver
2. https://github.com/kelseyhightower/coreos-ipxe-server
