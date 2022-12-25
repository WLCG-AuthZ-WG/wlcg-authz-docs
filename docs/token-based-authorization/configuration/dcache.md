# dCache

[Official documentation](https://dcache.org/old/manuals/Book-6.2/config-gplazma.shtml#using-openid-connect) currently doesn't provide all necessary information how to configured storage to accept and map WLCG JWT tokens (see [dCache#6607](https://github.com/dCache/dcache/issues/6607)). dCache provides two different gPlazma plugins to deal with OIDC tokens - `oidc` and `scitoken`. With dCache 7.x it is necessary to use `scitoken` plugin, because this is the only way to deal with WLCG JWT tokens or SCITOKENS. The dCache 8.x comes with more generic `oidc` plugin that can process also WLCG JWT tokens and SCITOKENS (`scitoken` plugin is still kept for compatibility reasons, but it is deprecated).

## WLCG JWT compliance testbed

There are several dCache instances already included in the [testbed](https://github.com/indigo-iam/wlcg-jwt-compliance-tests#storage-area-configuration-pre-requisites) including [`prometheus.desy.de`](https://confluence.desy.de/pages/viewpage.action?pageId=228761933), which is maitained by dCache developers and it is running `master` branch.

## dCache 7.2 configuration

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
# you can specify multiple audiences, but avoid using generic https://wlcg.cern.ch/jwt/v1/any on production instances
# (https://wlcg.cern.ch/jwt/v1/any is necessary for compliance testbed)
gplazma.scitoken.audience-targets = https://wlcg.cern.ch/jwt/v1/any https://dcache.example.com https://dcache.example.com:2880 https://alias.example.com
# ...
```

It is necessary to map identity extracted from WLCG JWT token (in this case we rely on multimap "`op`" that gives us access to the token issuer mapped to our name configured in the layout file) to the dCache `uid`, `gid` and `username`, e.g. by using multimap file `/etc/dcache/multi-mapfile.wlcg_jwt`
```
op:wlcg               uid:1999 gid:1999,true username:wlcg_oidc
op:atlas              uid:2999 gid:2999,true username:atlas_oidc
op:cms                uid:3999 gid:3999,true username:cms_oidc
```

Be wery careful how you map WLCG JWT token indentity and when you support also X.509 voms proxies. Most probably it'll be necessary to very carefully add additional ACLs to your VO (sub)directories.

## dCache 8.2 configuration

FIXME: this configuration was not yet tested
```
auth     optional     oidc
map      sufficient   multimap gplazma.multimap.file=/etc/dcache/multi-mapfile.wlcg_jwt
```
```
# ...
[centralDomain/gplazma]
# assuming that VO starts in top level directory
gplazma.oidc.provider!wlcg = https://wlcg.cloud.cnaf.infn.it/ -profile=wlcg -prefix=/wlcg
gplazma.oidc.provider!altas = https://atlas-auth.web.cern.ch/ -profile=wlcg -prefix=/atlas
gplazma.oidc.provider!cms = https://cms-auth.web.cern.ch/ -profile=wlcg -prefix=/cms
# assuming that dCache WebDAV service runs on default HTTPS port 443 for doors dcache.example.com
#gplazma.oidc.audience-targets = https://dcache.example.com
# you can specify multiple audiences, but avoid using generic https://wlcg.cern.ch/jwt/v1/any on production instances
# (https://wlcg.cern.ch/jwt/v1/any is necessary for compliance testbed)
gplazma.oidc.audience-targets = https://wlcg.cern.ch/jwt/v1/any https://dcache.example.com https://dcache.example.com:2880 https://alias.example.com
# ...
```

## ACL configuration

To be able to use ACLs with dCache it is necessary to enable them in the `dcache.conf`
```
# enable ACL support
pnfsmanager.enable.acl = true
```
Currently it is not possible to modify ACL recursively using `chimera` interface ([dCache#6844](https://github.com/dCache/dcache/issues/6844)), but you can use [NFS mounted dCache](https://dcache.org/old/manuals/Book-8.2/config-nfs.shtml) and `nfs4_setfacl`, e.g.
1. enable NFS doors service by updating layout configuration file, e.g.
```
[doorsDomain/nfs]
nfs.version = 4.1
```
2. export NFS filesystem: `echo "/ 127.0.0.1(rw,no_root_squash)" >> /etc/exports`
3. restart dCache: `systemctl restart dcache.target`
4. mount dCache: `mount -o acl,rw 127.0.0.1:/ /mnt`
5. list current ACL configuration: `nfs4_getfacl /mnt/atlas`
6. use listed ACI to recursively update ACL: `nfs4_setfacl -R -P -s A:fdg:3000:rx,A:fdg:3003:rwaDdx,A:fdg:3004:rwaDdx,A:fdg:3099:rwaDdx /mnt/atlas`

### WLCG compliance testbed

Storage [configuration](https://github.com/indigo-iam/wlcg-jwt-compliance-tests) for compliance tests.

### ATLAS
```
/atlas-root ... read only for /atlas group
/atlas-root/atlasscratchdisk ... inheritable read for FQAN:/atlas and write for FQAN:/atlas
/atlas-root/atlasdatadisk ... inheritable read for FQAN:/atlas and write for FQAN:/atlas/Role=production
/atlas-root/atlaslocalgroupdisk ... inheritable read for FQAN:/atlas and write for FQAN:/atlas/country_code
```
Example for following X.509 identity mapping `FQAN:/atlas` gid 2000, `FQAN:/atlas/Role=production` gid 2001, `FQAN:/atlas/cz` gid 2002
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
