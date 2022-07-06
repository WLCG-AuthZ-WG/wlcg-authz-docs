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
group:wlcg_oidc               uid:1100, gid:1100,true username:wlcg_oidc
group:atlas_oidc              uid:2000, gid:2000,true username:atlas_oidc
group:cms_oidc                uid:3000, gid:3000,true username:cms_oidc
```

Be wery careful when you map WLCG JWT token indentity when you decide to use it together with X.509 voms proxies. Most probably it'll be necessary to very carefully add additional ACLs to your VO (sub)directories.

## dCache 8.2 configuration
