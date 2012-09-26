===================
 Ceph training vms
===================

Welcome. For documentation on how to actually use the virtual
machines, please refer to your training material. This file is mostly
concerned with creating and updating the images.


How to build & publish images
=============================

To build the images, run::

  ./run

and hope for the best. The full toolchain required is not well
documented, at this point.

To upload, you need to have a properly-configured botosh_ with a
service named ``train`` configured, pointing to DreamObjects bucket
``inktank-training-vms``, see
http://objects.dreamhost.com/inktank-training-vms/index.html . To
perform the upload, run::

  ./upload

.. _botosh: https://github.com/tv42/botosh


There is no versioning on the images themselves, currently. Don't make
a mess.


Details about the virtualization
================================


File formats
------------

OVF_ is a standard interchange format for virtual machines, and
collections of virtual machines.

.. _OVF: http://www.dmtf.org/standards/ovf

OVA is essentially a tarball of the OVF and the disk images. It is
meant to simplify downloading the images, but does not seem to be as
widely supported.

The list of VMDK image subtypes was hard to find, so archiving here:
http://www.vmware.com/support/developer/vc-sdk/visdk41pubs/ApiReference/vim.OvfManager.CreateImportSpecParams.DiskProvisioningType.html


Hypervisor support
------------------

The images have been testd on VirtualBox_ on Linux. They are expected
to work on all VirtualBox platforms.

.. _VirtualBox: http://www.virtualbox.org/

VMWare's commercial offerings are expected to import the OVA/OVF
without problems. With VMWare Player, there are conflicting reports,
see http://superuser.com/questions/81129/running-ovf-on-vmware-player
for more. Also see
http://askubuntu.com/questions/153486/how-do-i-boot-ubuntu-cloud-images-in-vmware
.


You should be able to get the disk images to work with KVM/QEmu or
libvirt, but we are not aware of any OVF support, so you'll need to
define the virtual machines manually.


Workarounds
-----------

- VirtualBox does not support OVF Environments, so we can't configure
  cloud-init that way -> manually provide a disk image to configure
  the vm
- don't know what ``ovf:format`` to use for an ISO image -> use VFAT
  for cloud-init configuration image
- VirtualBox does not seem to support CoW parent images, when
  importing an OVF -> refer to the same File from multiple Disks
- referring to the same File from multiple Disks seems to break
  VirtualBox's OVA support -> for VirtualBox, download all the other
  files, except for the ``*.ova`` [#vbox10689]_
- VirtualBox does not seem to support blank disks, when importing an
  OVF -> need to include dummy VMDKs, even for empty disks
- VMDK support in QEmu/VirtualBox is quite buggy, yet it seems to be
  the one disk image format supported by all platforms -> jump through
  hoops with ``qemu-img`` and ``vboxmanage clonehd``, to get the VMDKs
  into ``streamOptimized`` format. [#ubuntu1006655]_
- VirtualBox has trouble verifying the manifests, this may be related
  to the CoW workarounds -> hope your downloads work right

.. [#vbox10689] https://www.virtualbox.org/ticket/10689

.. [#ubuntu1006655] https://bugs.launchpad.net/ubuntu/+source/qemu-kvm/+bug/1006655

