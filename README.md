# Cloud Instance Starter Kit

This project is intended to serve as a starting point for your own
Vagrant configuration. It will produce a fully updated VM that you can
connect to with `vagrant ssh`, without any manual intervention; if you
are using VirtualBox, the VirtualBox Guest Additions are installed
automatically as part of the bootstrap process.

## Are the VirtualBox Guest Additions necessary?

The VirtualBox [Guest Additions] provide a number of useful features,
but they are optional (although Vagrant will display a warning if they
are missing). The most important features are shared folders, and
improved time synchronization. They might also help with clock stability
problems on OS X hosts.

If you only need to share files between the host and the guest, please
consider using [NFS] instead of the VirtualBox Guest Additions: it's
significantly faster and it doesn't require additional maintenance
effort when VirtualBox itself or the guest kernel are updated.

[Guest Additions]: https://www.virtualbox.org/manual/ch04.html#idp46691714790384
[NFS]: https://www.vagrantup.com/docs/synced-folders/nfs.html

## Host requirements

Vagrant and Ansible should be installed on the host. I tested with
Vagrant 1.8.1 and Ansible 2.0.1.0 on OS X; while a different host OS
shouldn't create any problems, older Vagrant or Ansible versions might.

## Supported guest OSes

Only the following images are supported:

* `centos/7` (CentOS Linux 7, **default**)
* `centos/6` (CentOS Linux 6)
* `projectatomic/adb` (Atomic Developer Bundle)

## How to use

If you are using another operating system than OS X on the host, please
edit the `Vagrantfile` to provide the correct path to the
`VBoxGuestAdditions.iso` file (this should be part of your VirtualBox
installation).

If you are creating a new VM, just run `vagrant up`. The process takes
around 15 minutes on my old host (made in 2009) - it should be much
faster on modern hardware. When it's ready, you can connect to the new
instance with `vagrant ssh`.

If you later need to upgrade the guest additions, run `vagrant
provision`; this should only be necessary after a kernel update in the
VM, or after upgrading VirtualBox on the host.

## Known issues

* If you are getting an error while Ansible is waiting for the host to
  reboot, increase the `delay` parameter of the `wait_for` module in
  `provisioning/roles/base/handlers/main.yml`

* The guest will always reboot if it had to apply any updates, before
  installing the guest additions. This is only necessary if the kernel
  or some development headers were updated, but since this is not trivial
  to detect, we will always reboot - just to be sure.

* Vagrant sometimes fails to connect to the guest. This seems to be
  caused by the guest occasionally not getting a reply from the DHCP
  server in VirtualBox. After running `vagrant up; vagrant destroy -f` in
  a loop over night, I saw this problem in almost 10% of boot attempts.
  If this happens, you can log in from the console to shut the VM down
  (VirtualBox 5 allows you to attach to a headless VM).

* VirtualBox 5 provides a KVM paravirtualization interface by default
  for Linux guests, which causes the guest kernel to set its clocksource
  to kvm-clock. On some older processors, this causes the system time to
  significantly fall behind the real time when the CPU is idle (due to
  the TSC halting when the CPU is idle). Set the paravirtualization
  interface to `none` if you encounter this problem.
