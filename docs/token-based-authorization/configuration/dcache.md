# dCache

[Official documentation](https://dcache.org/old/manuals/Book-6.2/config-gplazma.shtml#using-openid-connect) currently doesn't provide all necessary information how to configured storage to accept and map WLCG JWT tokens (see [dCache#6607](https://github.com/dCache/dcache/issues/6607)). dCache provides two different gPlazma plugins to deal with OIDC tokens - `oidc` and `scitoken`. With dCache 7.x it is necessary to use `scitoken` plugin, because this is the only way to deal with WLCG JWT tokens or SCITOKENS. The dCache 8.x comes with more generic `oidc` plugin that can process also WLCG JWT tokens and SCITOKENS (`scitoken` plugin is still kept for compatibility reasons, but it is deprecated).

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
gplazma.scitoken.issuer!wlcg = https://wlcg.cloud.cnaf.infn.it/ /wlcg org.dcache.auth.GroupNamePrincipal:wlcg_oidc
gplazma.scitoken.issuer!altas = https://atlas-auth.web.cern.ch/ /atlas org.dcache.auth.GroupNamePrincipal:atlas_oidc
gplazma.scitoken.issuer!cms = https://cms-auth.web.cern.ch/ /atlas org.dcache.auth.GroupNamePrincipal:cms_oidc
# assuming that dCache WebDAV service runs on default HTTPS port 443 for doors dcache.example.com
gplazma.scitoken.audience-targets = https://dcache.example.com
# you can specify multiple audiences, but avoid using generic https://wlcg.cern.ch/jwt/v1/any on production instances
#gplazma.scitoken.audience-targets = https://wlcg.cern.ch/jwt/v1/any https://dcache.example.com https://dcache.example.com:1443 https://alias.example.com
# ...
```

Now it is necessary to map identity extracted from WLCG JWT token to the dCache `uid`, `gid` and `username`, e.g. by using multimap file `/etc/dcache/multi-mapfile.wlcg_jwt`
```
group:wlcg_oidc               uid:1999 gid:1999,true username:wlcg_oidc
group:atlas_oidc              uid:2999 gid:2999,true username:atlas_oidc
group:cms_oidc                uid:3999 gid:3999,true username:cms_oidc
```

Be wery careful when you map WLCG JWT token indentity when you decide to use it together with X.509 voms proxies. Most probably it'll be necessary to very carefully add additional ACLs to your VO (sub)directories.

## dCache 8.2 configuration

## ACL configuration

To be able to use ACLs with dCache it is necessary to enable them in the `dcache.conf`
```
# enable ACL support
pnfsmanager.enable.acl = true
```

### WLCG compliance testbed

Storage [configuration](https://github.com/indigo-iam/wlcg-jwt-compliance-tests) for compliance tests.

### ATLAS
```
/basepath/atlas ... read only for /atlas group
/basepath/atlas/atlasscratchdisk ... inheritable read for /atlas and write for /atlas
/basepath/atlas/atlasdatadisk ... inheritable read for /atlas and write for /atlas/Role=production
/basepath/atlas/atlaslocalgroupdisk ... inheritable read for /atlas and write for /atlas/country_code
```
Example for following X.509 identity mapping `/atlas` gid 2000, `/atlas/Role=production` gid 2001, `/atlas/cz` gid 2002
```
$ chimera ls /dpm/farm.particle.cz/home 
dr-xr-x---   19 2000 2000        512 May 10 00:00 atlas
$ chimera ls /dpm/farm.particle.cz/home/atlas 
drwxr-xr-x   10 2001 2001        512 Jun 01  2020 atlasdatadisk
drwxr-xr-x    5 2001 2001        512 Jan 18  2020 atlaslocalgroupdisk
drwxr-xr-x   60 2001 2001        512 Aug 19  2021 atlasscratchdisk
$ chimera getfacl /dpm/farm.particle.cz/home/atlas/atlasscratchdisk 
GROUP:2000:+lfsxDd:fdg
$ chimera getfacl /dpm/farm.particle.cz/home/atlas/atlasdatadisk 
GROUP:2001:+lfsxDd:fdg
GROUP:2000:+lx:fdg
$ chimera getfacl /dpm/farm.particle.cz/home/atlas/atlaslocalgroupdisk 
GROUP:2002:+lfsxDd:fdg
GROUP:2001:+lfsxDd:fdg
GROUP:2000:+lx:fdg
```
