# dCache

[Official documentation](https://dcache.org/old/manuals/Book-6.2/config-gplazma.shtml#using-openid-connect) currently doesn't provide all necessary information how to configured storage to accept and map WLCG JWT tokens (see [dCache#6607](https://github.com/dCache/dcache/issues/6607)). dCache provides two different gPlazma plugins to deal with OIDC tokens - `oidc` and `scitoken`. With dCache 7.x it is necessary to use `scitoken` plugin, because this is the only way to deal with WLCG JWT tokens or SCITOKENS. The dCache 8.x comes with more generic `oidc` plugin that can process also WLCG JWT tokens and SCITOKENS (`scitoken` plugin is still kept for compatibility reasons, but it is deprecated).

## WLCG JWT compliance testbed

There are several dCache instances already included in the [testbed](https://github.com/indigo-iam/wlcg-jwt-compliance-tests#storage-area-configuration-pre-requisites) including [`prometheus.desy.de`](https://confluence.desy.de/pages/viewpage.action?pageId=228761933), which is maitained by dCache developers and it is running `master` branch.

## dCache plugins for WLCG/SciToken profile

### dCache 7.2 configuration

To accept connection authorized with WLCG JWT tokens it is necessary to add `scitoken` plugin in the chain of gPlazma auth plugins. Just add following line in your `/etc/dcache/gplazma.conf` configuration
```
auth     optional     scitoken
map      sufficient   multimap gplazma.multimap.file=/etc/dcache/multi-mapfile.wlcg_jwt
```

This line alone is not sufficient for gPlazma service configuration, because it is necessary to specify supported WLCG JWT issuers. This can be done in the corresponding layout file with defined gPlazma service e.g.
```
# ...
[centralDomain/gplazma]
# assuming that VO starts in top level directory
gplazma.scitoken.issuer!wlcg = https://wlcg.cloud.cnaf.infn.it/ /wlcg
gplazma.scitoken.issuer!altas = https://atlas-auth.web.cern.ch/ /atlas
gplazma.scitoken.issuer!cms = https://cms-auth.web.cern.ch/ /cms
# assuming that dCache WebDAV service runs on default HTTPS port 443 for doors dcache.example.com
#gplazma.scitoken.audience-targets = https://dcache.example.com
# you can specify multiple audiences (https://wlcg.cern.ch/jwt/v1/any is necessary for compliance testbed)
gplazma.scitoken.audience-targets = https://wlcg.cern.ch/jwt/v1/any https://dcache.example.com https://dcache.example.com:2880 https://alias.example.com roots://dcache.example.com 
# ...
```

It is necessary to map identity extracted from WLCG JWT token (in this case we rely on multimap "`op`" that gives us access to the token issuer mapped to our name configured in the layout file) to the dCache `uid`, `gid` and `username`, e.g. by using multimap file `/etc/dcache/multi-mapfile.wlcg_jwt`
```
op:wlcg               uid:1999 gid:1999,true username:wlcg_oidc
op:atlas              uid:2999 gid:2999,true username:atlas_oidc
op:cms                uid:3999 gid:3999,true username:cms_oidc
```

Be wery careful how you map WLCG JWT token indentity and when you support also X.509 voms proxies. Most probably it'll be necessary to very carefully add additional ACLs to your VO (sub)directories or rely on dCache ownership inheritance from the parent directory.

### dCache 8.2 configuration
This version comes with

1. WLCG JWT profile support integrated in "oidc" gPlazma plugin
2. Support for access with tokens with roots:// protocol on same port 1094
3. Starting with 8.2.7 preferred authorization plugin is set to ZTN and we can simply rely on BEARER_TOKEN variable
4. Symlinks properly handled since 8.2.22

Following configuration updates should add support for token access
```
# /etc/dcache/gplazma.conf
...
auth     optional     oidc
map      sufficient   multimap gplazma.multimap.file=/etc/dcache/multi-mapfile.oidc
...
```
```
# /etc/dcache/layouts/your_layout_file.conf
# ...
[centralDomain/gplazma]
# assuming that VO starts in top level directory
gplazma.oidc.provider!wlcg = https://wlcg.cloud.cnaf.infn.it/ -profile=wlcg -prefix=/wlcg
gplazma.oidc.provider!altas = https://atlas-auth.web.cern.ch/ -profile=wlcg -prefix=/atlas
gplazma.oidc.provider!cms = https://cms-auth.web.cern.ch/ -profile=wlcg -prefix=/cms
# assuming that dCache WebDAV service runs on default HTTPS port 443 for doors dcache.example.com
#gplazma.oidc.audience-targets = https://dcache.example.com
# you can specify multiple audiences (https://wlcg.cern.ch/jwt/v1/any is necessary for compliance testbed)
gplazma.oidc.audience-targets = https://wlcg.cern.ch/jwt/v1/any https://dcache.example.com https://dcache.example.com:2880 https://alias.example.com
# ...
```
```
# /etc/dcache/multi-mapfile.oidc
op:wlcg               uid:1999 gid:1999,true username:wlcg_oidc
op:atlas              uid:2999 gid:2999,true username:atlas_oidc
op:cms                uid:3999 gid:3999,true username:cms_oidc
```

## Supporting X.509 and token access at same time

### dCache ownership inheritance

Alternatively to the ACL configuration it is also possible to rely on simple dCache ownership inheritance. Configuration option [`pnfsmanager.enable.inherit-file-ownership`](https://github.com/dCache/dcache/blob/8.2.11/skel/share/defaults/pnfsmanager.properties#L136-L149) can be used to force dCache to assign parent directory owner and group to all new files and subdirectories. This means objects get same owner and groups regardless of client authorization method (access with X.509 VOMS proxy or tokens).

### ACL configuration

To be able to use ACLs with dCache it is necessary to enable them in the `dcache.conf`
```
# enable ACL support
pnfsmanager.enable.acl = true
```

#### Recursive ACL updates

Currently it is not possible to modify ACL recursively using `chimera` interface ([dCache#6844](https://github.com/dCache/dcache/issues/6844)), but you can use [NFS mounted dCache](https://dcache.org/old/manuals/Book-8.2/config-nfs.shtml) and `nfs4_setfacl`, e.g.
1. enable NFS doors service by updating layout configuration file, e.g.
```
[doorsDomain/nfs]
nfs.version = 4.1
```
2. export NFS filesystem: `echo "/ 127.0.0.1(rw,no_root_squash)" >> /etc/exports`
3. restart dCache: `systemctl restart dcache.target`
4. mount dCache: `mount -o acl,rw 127.0.0.1:/ /mnt`
5. list current ACL configuration:
```
nfs4_getfacl /mnt/atlas/atlaslocalgroupdisk
A:fdg:2000:rx
A:fdg:2001:rwaDdx
A:fdg:2002:rwaDdx
A::OWNER@:rwaDxtTcC
A::GROUP@:rxtc
A::EVERYONE@:rxtc
```
6. use listed ACI to recursively update ACL:
```
nfs4_setfacl -R -P -s A:fdg:2000:rx,A:fdg:2001:rwaDdx,A:fdg:2002:rwaDdx,A:fdg:2099:rwaDdx /mnt/atlas/atlaslocalgroupdisk
```
7. unmount dCache: `umount /mnt`

## Supported authorizations

1. Issuer based authorization - access can be granted for any token from given issuer regardless of token content by maping `op:isser`
2. Group based authorization - when dCache decodes groups from the token (e.g. from `wlcg.groups` claim in case of WLCG profile) mapping can be applied to `oidcgrp:/vo/group_name`
3. Scope based authorization - dCache supports access based on storage capabilities defined in several OIDC profiles (e.g. WLCG & SciToken) 

It is not straightforward to configure dCache properly to use just scope based authorization. WLCG experiments prefers capability security model when it comes to the storage access and that means access with just issuer or group based authorization should be denied. Current dCache releases can't be directly configured to allow access only to one of listed authorization methods and it is necessary to apply special mapping in order to achieve desired behavior.

dCache OIDC plugin configured to use WLCG profile accepts additional configuration parameter `authz-id` which is used only for tokens that came with explicit AuthZ statements in the scope claim (`storage.*:/` scope). This behavior enables us to differentiate clients later during identity mapping based on the authorizations available in the token. Clients that did not came with scope based authorization will not be mapped to the dCache `uid` and `gid` and/or their namespace root can be set to non-existing directory. Wrong mapping leads to rejected client request.

Following changes needs to be applied to the dCache token configuration described earlier
* gPlazma OIDC provider configuration must be configured with `authz-id` parameter
```
# /etc/dcache/layouts/your_layout_file.conf
# ...
[centralDomain/gplazma]
# assuming that VO starts in top level directory
gplazma.oidc.provider!wlcg = https://wlcg.cloud.cnaf.infn.it/ -profile=wlcg -prefix=/wlcg -authz-id=username:wlcg_oidc_with_storage_scope
# ...
```
* new multimap mapping for unsupported OIDC authorization methods
```
# /etc/dcache/gplazma.conf
...
auth     optional     oidc
map      sufficient   multimap gplazma.multimap.file=/etc/dcache/multi-mapfile.oidc
map      sufficient   multimap gplazma.multimap.file=/etc/dcache/multi-mapfile.oidc-noop
...
session  sufficient   omnisession
#session optional     authzdb
...
```
* content of multimap files ensure access only to the clients that use tokens with explicit `storage.*` authorization
```
# /etc/dcache/multi-mapfile.oidc
username:wlcg_oidc_with_storage_scope     uid:1999 gid:1999,true group:writer
```
```
# /etc/dcache/multi-mapfile.oidc-noop
# intentionally wrong/insufficient mapping with missing dCache uid and gid
op:wlcg               username:noop group:noop
```
* assign non-existing root for tokens without storage scopes
```
# /etc/dcache/omnisession.conf
group:noop            read-only root:/noop home:/
```
In case dCache still rely on old `authzdb` plugin than namespace root can be set in `/etc/grid-security/storage-authzdb` to the non-existing directory, e.g.
```
# /etc/grid-security/storage-authzdb
authorize noop read-only 1999 1999 / /noop /
```

## Storage mapping and permissions configuration

### WLCG compliance testbed

Storage [configuration](https://github.com/indigo-iam/wlcg-jwt-compliance-tests) for compliance tests.

### ATLAS

With ***dCache ownership inheritance*** the configuration is straightforward and it is sufficient to set [right ownership](https://twiki.cern.ch/twiki/bin/viewauth/AtlasComputing/StorageSetUp#Recommendation) only to the top level directories in the root of ATLAS namespace. Most common configuration can be very simple
* `/atlas-root` - read only for /atlas group
* `/atlas-root/atlasscratchdisk` - inheritable read for `FQAN:/atlas` and write for `FQAN:/atlas`
* `/atlas-root/atlasdatadisk` - inheritable read for `FQAN:/atlas` and write for `FQAN:/atlas/Role=production`
* `/atlas-root/atlaslocalgroupdisk` - inheritable read for `FQAN:/atlas` and write for `FQAN:/atlas/country_code`

When dCache rely just on ***ACLs the configuration*** can be slightly more complex, for example following X.509 identity mapping `FQAN:/atlas` gid 2000, `FQAN:/atlas/Role=production` gid 2001, `FQAN:/atlas/cz` gid 2002 could be used to achieve behavior expected from ATLAS storage
```
$ chimera ls /
dr-xr-x---   19 2000 2000        512 May 10 00:00 atlas-root
$ chimera ls /atlas-root
drwxr-xr-x   10 2001 2001        512 Jun 01  2020 atlasdatadisk
drwxr-xr-x    5 2001 2001        512 Jan 18  2020 atlaslocalgroupdisk
drwxr-xr-x   60 2001 2001        512 Aug 19  2021 atlasscratchdisk
$ chimera getfacl /atlas-root/atlasscratchdisk
GROUP:2000:+lfsxDd:fdg
$ chimera getfacl /atlas-root/atlasdatadisk
GROUP:2001:+lfsxDd:fdg
GROUP:2000:+lx:fdg
$ chimera getfacl /atlas-root/atlaslocalgroupdisk
GROUP:2002:+lfsxDd:fdg
GROUP:2001:+lfsxDd:fdg
GROUP:2000:+lx:fdg
```
Both configurations works well with clients accessing storage either with X.509 or WLCG tokens and scope based authorization.

#### configuration
```
# /etc/dcache/layouts/your_layout_file.conf
# ...
[centralDomain/gplazma]
# assuming that VO starts in top level directory /atlas
gplazma.oidc.provider!atlas = https://atlas-auth.web.cern.ch/ -profile=wlcg -prefix=/atlas -authz-id=username:atlas_oidc_with_storage_scope
# you can specify multiple audiences, e.g. https://wlcg.cern.ch/jwt/v1/any is necessary
# for compliance testbed and also CMS, but for ATLAS it is currently sufficient to use
# "headnode" (doors) hostname(s) / hostnames defined in the ATLAS Rucio protocol configuration
# that can be obtained with `rucio-admin rse info PRAGUELCG2_DATADISK` or from ATLAS CRIC web interface
gplazma.oidc.audience-targets = davs.example.com xroot.example.com
# ...
```
```
# /etc/dcache/gplazma.conf
...
auth     optional     oidc
map      sufficient   multimap gplazma.multimap.file=/etc/dcache/multi-mapfile.oidc
map      sufficient   multimap gplazma.multimap.file=/etc/dcache/multi-mapfile.oidc-noop
...
# this depends if your dCache still rely on storage-authzdb or omnisession as described in earlier sections
session  sufficient   omnisession
#session optional     authzdb
...
```
```
# /etc/dcache/multi-mapfile.oidc
# This mapping applies to the tokens that came with explicit AuthZ
# statements in the scope claim (storage.*:/ scopes in case of
# WLCG/SciTokens).
username:atlas_oidc_with_storage_scope     uid:2000 gid:2001,true
# sites that used DPM to dCache migration tools should add group:writer
#username:atlas_oidc_with_storage_scope    uid:2000 gid:2001,true group:writer
```
```
# /etc/dcache/multi-mapfile.oidc-noop
# Tokens with just group (oidcgrp:) or issuer (op:) based authorization
# are not mapped in the multi-mapfile.oidc, because dCache doesn't add
# principals from authz-id parameter defined with gplazma.oidc.provider
#
# If we would like to support just capability based access with WLCG/SciToken
# profile (tokens with storage.*:/ scopes) we intentionally use bad (sufficient)
# mapping configured here to deny access. Although missing dCache uid and gid
# mapping is sufficient to reject clients special group "noop" is also mapped
# to non-existing directory in the omnisession or storage-authzdb configuration file.
op:atlas              username:noop group:noop
```
```
# /etc/dcache/omnisession.conf
...
# deny access by mapping "noop" group to non-existing root directiory
# this is here just for safety, because client should be already kicked
# out while its identity is not even mapped to any dCache uid and gid
group:noop            read-only root:/noop home:/
...
```
