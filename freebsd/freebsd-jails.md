# freebsd jails

work in progress

freebsd thin jails for services

* switch to root

```
sudo su
```

create a zfs dataset for jails and set the mountpoint

```
zfs create -o mountpoint=/usr/local/jails zroot/jails
```

create a dataset for the thinjails and the name of the release you are going to download

```
zfs create -p zroot/jails/thinjails
zfs create -p zroot/jails/releases/11.2-RELEASE
```

## download the base system tarball

download base system tarball,  
change the url from 11.2-STABLE to the release version you want

```
fetch http://ftp.freebsd.org/pub/FreeBSD/snapshots/amd64/11.2-STABLE/base.txz -o /tmp/base.txz
```

use the built in fetch command to download the file rather than wget or curl which arent installed by default  
save the download file to /tmp


* Extract the base sytem from /tmp to the 11.2-RELEASE directory

```
tar -xf /tmp/base.txz -C /usr/local/jails/releases/11.2-RELEASE
```

Make sure you jail has the right timezone and dns servers

```
cp /etc/resolv.conf /usr/local/jails/releases/11.2-RELEASE/etc
cp /etc/localtime /usr/local/jails/releases/11.2-RELEASE/etc/localtime
```

### update freebsd base install

Run freebsd update on the basejail system.

```
freebsd-update -b /usr/local/jails/releases/11.2-RELEASE fetch install
```

* verify the freebsd checksum

```
freebsd-update -b /usr/local/jails/releases/11.2-RELEASE IDS
```

## jail template


* Create a zfs snapshot.

```
zfs snapshot zroot/jails/releases/11.2-RELEASE@template
```

use zfs send to send the 11.2-RELEASE snapshot to a new base jail.  
This will be the first layer in future jails.

```
zfs create -p zroot/jails/releases/11.2-RELEASE
zfs send -R zroot/jails/releases/11.2-RELEASE@template | zfs recv zroot/jails/templates/base-11.2-RELEASE
```

Now, some tutorials suggest ZFS cloning (ie. zfs clone). This in itself indeed is the most simple way to clone a basejail to a production jail, however ZFS clones have certain drawbacks which over time completely negate any benefits. A ZFS clone is basically a snapshot, the filesystem is not physically duplicated and it saves all that space (a base 10.3 jail is some 300MB large). At first. Because as you use the jail and update it, more and more of those files are copied to individual jails as they change. Also, you cannot destroy a snapshot which has existing clones. That means you'd have to keep around all the basejail snapshots, or "promote" cloned jails. So why not just send | receive and copy the basejail which is independent from the start. 


### jail skeleton

In addition to your base template, you need to create a skeleton template which will hold all the directories that are local to your jail.  
We’re going to copy these directories from the template to the skeleton.

* Create a skeleton dataset that will be used for jail-specific files

```
zfs create -p zroot/jails/templates/skeleton-11.2-RELEASE
```

* move and symlink

The skeleton directory is what is going to be copied for each new jail. It is going to be mounted in /skeleton/ inside the jail. So in the read-only base template we need to create symlink from all the expected locations to the appropriate directories inside the /skeleton/ directory. It is very important to cd into your jail directory and create these symlinks with relative paths. That way they will always link to the correct location no matter where the base template ends up mounted.

#### create new thinjail

Configure the jail hostname.  
the name will be thinjail1

```
echo hostname=\"thinjail1\" > /usr/local/jails/releases/11.2-RELEASE/etc/rc.conf
```

Make the jail directory where the base template and skeleton folder will be mounted.

```
mkdir -p /usr/local/jails/thinjail1
```

* Create the jail entry in /etc/jail.conf, be sure and include the global jail configs listed in the fulljail example.

```
# The jail definition for thinjail1
thinjail1 {
    host.hostname = "thinjail1.domain.local";
    path = "/usr/local/jails/thinjail1";
    interface = "lagg0";
    ip4.addr = 10.0.0.17;
    mount.fstab = "/usr/local/jails/thinjail1.fstab";
}
```

* Create the jail fstab.

```
# /usr/local/jails/thinjail1.fstab

/usr/local/jails/templates/base-11.2-RELEASE  /usr/local/jails/thinjail1/ nullfs   ro          0 0
/usr/local/jails/thinjails/thinjail1     /usr/local/jails/thinjail1/skeleton nullfs  rw  0 0
```

## host rc.conf

First off we need a new loopback network interface to communicate over, so we should add the following string to /etc/rc.conf:

```
jail_enable="YES"
cloned_interfaces="lo1"
ifconfig_lo1_alias0="inet 172.16.1.1 netmask 255.255.255.0"

# If you need more IP addresses for jails in the future, add
# another line here like
# ifconfig_lo1_alias1="inet 172.16.1.2 netmask 255.255.255.0"
```

This creates a new network interface named lo1 which is given an IP address of 172.16.1.1.  
You can give it a different IP address, but make sure that it’s one of the RFC 1918 private IP addresses.

These network settings will have to be loaded after you save your edits to /etc/rc.conf,  
so you can either restart you machine or run the equivalent ifconfig commands to setup the new interface on a running system.

```
sudo ifconfig lo1 create
sudo ifconfig lo1 alias 172.16.1.1 netmask 255.255.255.0
```

### jail.conf

```
# /etc/jail.conf

# Global settings applied to all jails.

exec.start = "/bin/sh /etc/rc";
exec.stop = "/bin/sh /etc/rc.shutdown";
exec.clean;
mount.devfs;

# The name of each jail. $name is a placeholder.
host.hostname = "${name}.domain.local";

# path to the jail
path = "/usr/local/jails/${name}";

# The IP address of the jail.
ip4.addr = 172.16.1.${ip};

# Jail definition for thinjail1.
thinjail1 {
   $ip = 1;
}
```

### pf firewall

```
# /etc/pf.conf

ext_if = "vtnet0"
ext_addr = $ext_if:0
int_if = "lo1"
jail_net = $int_if:network

nat on $ext_if from $jail_net to any -> $ext_addr port 1024:65535 static-port 
```

### start the jail

```
jail -c thinjail1
```

You can open a shell within the jail using

```
jexec thinjail1 sh
```

## creating new jails

For each new jail, just follow the process:

Add config to /etc/jail.conf
Clone skeleton to /usr/local/jails/thinjails/<jailname>
Write hostname to /etc/rc.conf in new jail files
Create folder /usr/local/jails/<jailname> for new jail
Create fstab /usr/local/jails/<jailname>.fstab and populate with layer information
Create and start with jail -c jailname


### upgrading thinjails

Whenever we want to update all jails at once, shut down the jails

Minor upgrades to a basejail can easily be done via freebsd-update:

```
freebsd-update -b /path/to/basejail fetch
freebsd-update -b /path/to/basejail install
```

Upgrades between major releases are best handled differently: Simply create a new basejail for the new system version and link the new basejail into the existing jail in place of the usual one:

Stop the jail.  
Change the mount = ... option in jail.conf from the old to the new base jail

Upgrade the existing system configuration files in the jails using mergemaster:

```
mergemaster -F -t /path/to/actual/jail/var/tmp/temproot -D /path/to/actual/jail
```

Start the jail
Upgrade the packages if necessary (pkg-static install -f pkg, pkg upgrade)

Note: Upgrading the base-system often involves changes to files like passwd or group. It is advised to regenerate the databases associated to these files (and mergemaster can actually run those commands at the end of an upgrade), which can be done inside the jail using the following commands:

```
cap_mkdb /etc/login.conf
services_mkdb -q -o /var/db/services.db /etc/services
pwd_mkdb -d /etc -p /etc/master.passwd
```

### references

* [freebsd jails the hard way](https://clinta.github.io/freebsd-jails-the-hard-way/)
* [an introduction to jails](https://www.skyforge.at/posts/an-introduction-to-jails-and-jail-networking/)
* [freebsd thin jails](https://jacob.ludriks.com/2017/06/07/FreeBSD-Thin-Jails/)
