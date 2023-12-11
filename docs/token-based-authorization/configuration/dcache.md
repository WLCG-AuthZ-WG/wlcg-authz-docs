# dCache

[OIDC plugin reference](https://www.dcache.org/manuals/Book-9.2/config-gplazma.shtml#oidc) describes parameters and configuration options that can be used to deal with clients authorized with tokens. dCache supports several token profiles including WLCG/SciToken profiles. dCache provides two different gPlazma plugins to deal with OIDC tokens that follow WLCG profile - `oidc` and `scitoken`. With dCache 7.x it is necessary to use `scitoken` plugin, because this is the only way to deal with WLCG JWT tokens or SciTokens. The dCache 8.x comes with more generic `oidc` plugin that can process also WLCG JWT tokens and SciTokens (`scitoken` plugin is still kept for compatibility reasons, but it is deprecated).

## WLCG JWT compliance testbed

There are several dCache instances already included in the [testbed](https://github.com/indigo-iam/wlcg-jwt-compliance-tests#storage-area-configuration-pre-requisites) including [`prometheus.desy.de`](https://confluence.desy.de/pages/viewpage.action?pageId=228761933), which is maitained by dCache developers and it is running `master` branch.

## dCache plugins for WLCG/SciToken profile

### dCache 7.2 configuration

**WARNING**: LHC experiments don't consider dCache 7.x for transfers with tokens.

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

Be careful how you map WLCG JWT token indentity when you support also X.509 voms proxies and your dCache rely on ACLs for VO (sub)directories. WLCG JWT tokens allows access based on [scope or group authorization](https://github.com/WLCG-AuthZ-WG/common-jwt-profile/blob/master/profile.md#authorization) and specific requirements wre not considered in the simple mentioned above. The following sections provide some ideas to consider.

### dCache 8.2/9.2 configuration
These dCache versions comes with important updates.

1. dCache 8.2 comes with WLCG JWT profile support integrated in "oidc" gPlazma plugin and `root://` + `roots://` protocol configured on the same port
2. Starting with 8.2.7 preferred authorization plugin is set to ZTN and we can simply rely on BEARER_TOKEN variable
3. Symlinks properly handled since 8.2.22
4. WLCG JWT explicit authorization implemented in 8.2.32 and 9.2.0 (needs workaround in IAM token issuer)
5. Recommended for WLCG experiments are 8.2.35+ and 9.2.3+ (versions older than 8.2.22 can't be used with WLCG JWT tokens)

Following minimal configuration adds support to access files with WLCG JWL tokens
```
# /etc/dcache/gplazma.conf
...
auth     optional     oidc
...
```
```
# /etc/dcache/layouts/layout_file.conf
# ...
[centralDomain/gplazma]
# assuming that VO starts in top level directory
gplazma.oidc.provider!wlcg = https://wlcg.cloud.cnaf.infn.it/ -profile=wlcg -prefix=/wlcg -authz-id="uid:1999 gid:1999 username:wlcg_oidc"
gplazma.oidc.provider!altas = https://atlas-auth.web.cern.ch/ -profile=wlcg -prefix=/atlas -authz-id="uid:2999 gid:2999 username:atlas_oidc"
gplazma.oidc.provider!cms = https://cms-auth.web.cern.ch/ -profile=wlcg -prefix=/cms -authz-id="uid:3999 gid:3999 username:cms_oidc"
# assuming that dCache WebDAV service runs on default HTTPS port 443 for doors dcache.example.com
#gplazma.oidc.audience-targets = https://dcache.example.com
# you can specify multiple audiences (https://wlcg.cern.ch/jwt/v1/any is necessary for compliance testbed)
gplazma.oidc.audience-targets = https://wlcg.cern.ch/jwt/v1/any https://dcache.example.com https://dcache.example.com:2880 https://alias.example.com roots://dcache.example.com:1094 dcache.example.com ARBITRARY_STRING_REQUESTED_BY_EXPERIMENT
# ...
```
This configuration is sufficient for VOs that rely exclusively on scope based authorization and dCache which is not configured with ownership inheritance create new files with defined `uid` and `gid` number.

Access to the storage is restricted the the tokens with `aud` claim that match one of configured `gplazma.oidc.audience-targets`. It is up to experiments (VOs) to create tokens with desired audience and that's why one configuration for multiple VOs doesn't really weaken security for individual experiments.

## Supporting X.509 and token access at same time

There are (at least) two different ways that ensure consistent access for both authorization methods.

### dCache ownership inheritance

Configuration option [`pnfsmanager.enable.inherit-file-ownership`](https://github.com/dCache/dcache/blob/8.2.36/skel/share/defaults/pnfsmanager.properties#L159-L172) can be used to force dCache to assign parent directory owner and group to all new files and subdirectories. This means objects get same owner and groups regardless of client authorization method (access with X.509 VOMS proxy or tokens).

### ACL configuration

Alternatively to the dCache ownership inheritance it is also possible to rely on ACLs. ACL support must be first enabled in the `dcache.conf`
```
# enable ACL support
pnfsmanager.enable.acl = true
```
Default ACLs are also automatically inherited by newly created objects and this functionality is essential for ensuring consistent access for clients authorized with X.509 certificates or tokens.

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

WLCG experiments prefers capability security model when it comes to the storage access and that means access with just issuer or group based authorization should be denied. dCache OIDC plugin configured to use WLCG profile (selected by `-profile=wlcg` parameter) accepts additional configuration parameter `authz-id` which is used only for tokens that came with explicit AuthZ statements in the scope claim (e.g. `storage.read:/foo` scope). This parameter can be used to map directly token issuer to the dCache `uid`, `gid` and `username`. Clients that did not came with scope based authorization will not get mapping defined by `authz-id` and they'll not be authorized to access dCache storage. In case supported VO rely on issuer/group based authorization you could use `non-authz-id` parameter in the `gplazma.oidc.provider` configuration or later map `op:provider` and/or `oidcgrp:/vo/group` with `multimap` plugin. Wrong/missing mapping as usually leads to rejected client request.

Following changes needs to be applied to the dCache configuration to allow access for tokens with both, scope + group based authorization
* gPlazma OIDC provider configuration must be configured with `authz-id` parameter
```
# /etc/dcache/layouts/layout_file.conf
# ...
[centralDomain/gplazma]
# assuming that VO starts in top level directory
gplazma.oidc.provider!wlcg = https://wlcg.cloud.cnaf.infn.it/ -profile=wlcg -prefix=/wlcg -authz-id="username:atlas_oidc_scope_based uid:1000 gid:1000" -non-authz-id="username:atlas_oidc_group_based uid:1100 gid:1100"
# ...
```
* new multimap mapping for WLCG groups defined in access token
```
# /etc/dcache/gplazma.conf
...
auth     optional     oidc
map      sufficient   multimap gplazma.multimap.file=/etc/dcache/multi-mapfile.oidc
...
session  sufficient   omnisession
#session optional     authzdb
...
```
* content of multimap files to map additional groups
```
# /etc/dcache/multi-mapfile.oidc
# you could define access for tokens that don't even come with any WLCG group using issuer
#op:wlcg              gid:1200
oidcgrp:/wlcg         gid:1300
oidcgrp:/wlcg/test    gid:1400
```
In case dCache still rely on old `authzdb` plugin than namespace root can be set in `/etc/grid-security/storage-authzdb`, but you would have to create new group in the `gplazma.oidc.provider` configuration (e.g. `group:atlas_oidc`) or in the identity mapfiles to be able to set
```
# /etc/grid-security/storage-authzdb
authorize atlas_oidc 1000 1000 / / /
```
(not tested, author of this document have no sufficient experience with this deprecated module)

## Storage mapping and permissions configuration

### WLCG compliance testbed

Storage [configuration](https://github.com/indigo-iam/wlcg-jwt-compliance-tests) for compliance tests.

### ATLAS

ATLAS rely exclusively on scope based authorization, never include any mapping for `op:atlas` or `oidcgrp:/atlas` in your dCache configuration files.

With ***dCache ownership inheritance*** the configuration is straightforward and it is sufficient to set [right ownership](https://twiki.cern.ch/twiki/bin/viewauth/AtlasComputing/StorageSetUp#Recommendation) only to the top level directories in the root of ATLAS namespace. Most common configuration can be very simple

* `/atlas-root` - read only for /atlas group
* `/atlas-root/atlasscratchdisk` - inheritable read for `FQAN:/atlas` and write for `FQAN:/atlas`
* `/atlas-root/atlasdatadisk` - inheritable read for `FQAN:/atlas` and write for `FQAN:/atlas/Role=production`
* `/atlas-root/atlaslocalgroupdisk` - inheritable read for `FQAN:/atlas` and write for `FQAN:/atlas/country_code`

The configuration can be slightly more complex when dCache rely just on ***ACLs***, because then it is necessary to set directory permissions that will be inherited by new directories and files. For example following X.509 identity mapping `FQAN:/atlas` gid 2000, `FQAN:/atlas/Role=production` gid 2001, `FQAN:/atlas/cz` gid 2002 could be used to achieve behavior expected from ATLAS storage
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
# /etc/dcache/layouts/layout_file.conf
# ...
# use your own gplazma domain name (this example assumes gPlazma runs in centralDomain)
[centralDomain/gplazma]
# VO issuer prefix:
# assuming that namespace for VO data is stored in the top level directory /atlas
gplazma.oidc.provider!atlas = https://atlas-auth.web.cern.ch/ -profile=wlcg -prefix=/atlas -authz-id="uid:2001 gid:2001 username:atlas_oidc_with_storage_scope"
# In case ATLAS VO namespace starts in /pnfs/example.com/atlas than you must use this full prefix
# in the provider configuration. Using "/" prefix (most probably for any VO) is wrong with severe
# security implications
#
# Audience:
# audience contains union of `aud` claims used by all supported VOs for ATLAS it is currently
# sufficient to use "headnode" (doors) hostname(s) / hostnames defined in the ATLAS Rucio protocol
# configuration (see `rucio-admin rse info PRAGUELCG2_DATADISK` or ATLAS CRIC web interface
# https://atlas-cric.cern.ch/core/storageresource/list/), but to be compliant also with WLCG JWT profile
# requirements (which can become also ATLAS Rucio requirements in the future) for protocols
# davs://davs.example.com:443/atlas/data and root://xroot.example.com:1094/atlas/data
# you should use at least following configuration
gplazma.oidc.audience-targets = https://davs.example.com roots://xroot.example.com:1094 davs.example.com xroot.example.com
# ...
```
```
# /etc/dcache/gplazma.conf
...
auth     optional     oidc
...
```
