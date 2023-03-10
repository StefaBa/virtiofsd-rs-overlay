# virtiofsd-rs-overlay
### Gentoo overlay for the rust implementation of virtiofsd

ebuild files are automtically generated using [cargo-ebuild](https://github.com/gentoo/cargo-ebuild) by a scheduled github action.
So new versions in upstream should be included here very quickly.

**Currently no Manifests are shipped with this repo and `use-manifests` is set to `false`**

-> if this bothers you create an issue and i'll make manifest files.

(don't see the reason of maintaining those if i'm the only one using this repo)

---
create `/etc/portage/repos.conf/virtiofsd-rs-overlay.conf` with

    [virtiofsd-rs-overlay]
    location = /var/db/repos/virtiofsd-rs-overlay
    sync-type = git
    sync-uri = https://github.com/StefaBa/virtiofsd-rs-overlay.git
    auto-sync = yes

---
### Issues with the c implementation
The default c implementation of virtiofsd, shipped with current versions of qemu,
has a bug which doesn't allow the use of the xattrmap feature in a sanefull way.

For Example `virtiofsd xattrmap=:map::user.virtiofs.:`
should prepend user.virtiofs to all xattrs from the guest vm,
s.t. security.selinux labels are not stored as is on the host machine.
But if you use this feature with the c version of virtiofsd various file-operations
break on the guest vm (e.g. chown, chcon, chmod)

The rust implementation does not have this issue.

---
### Remarks regarding using a custom virtiofsd in libvirt
You should create a custom script in e.g. `/usr/local/bin/virtiofsd-xattrmap.sh` with the following contents:

    #!/bin/bash
    /usr/bin/virtiofsd-rs "$@",xattrmap=:map::user.virtiofs.:,security_label
    
as libvirt does not support specifying the xattrmap argument in it's XML config (possibly add posix-acl if you want support for posix acls. see also `virtiofsd-rs --help`).

And then, inside the libvirt XML, modify the binary path of the virtiofsd binary as such:
    
    <filesystem type='mount' accessmode='passthrough'>
      <driver type='virtiofs' queue='1024'/>
      <binary path='/usr/local/bin/virtiofsd-xattrmap.sh' xattr='on'/>
      <source dir='/some/dir'/>
      <target dir='some_label'/>
      <address type='pci' domain='0x0000' bus='0x01' slot='0x00' function='0x0'/>
    </filesystem>

---

