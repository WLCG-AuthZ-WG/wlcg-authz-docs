# XRootD

Token-based authentication and authorization is supported by [XRootD][xrootd]
in combination with the [xrootd-scitokens][xrootd-scitokens] plugin.

## Enabling token-based authN/Z for the WLCG IAM instance

In order to enable token-based AuthN/Z for the WLCG IAM instance on an XRootD
deployment, you need to:

- Install the `xrootd-scitokens` plugin. This is for example part
  of the upstream XRootD yum repository.

- Add the WLCG issuer to the list of the trusted issuers by the SciTokens
  library. This can be done by creating a configuration file e.g. at
  `/etc/xrootd/scitokens.cfg` with the following content
  (assuming `xrootd` should act as the `xrootd` user and files stored
  in the `/data/grid`):
```
[Global]
onmissing = passthrough
audience = https://wlcg.cern.ch/jwt/v1/any, https://xrd.example.com:1094, roots://xrd.example.com:1094

[Issuer WLCG IAM]
issuer = https://wlcg.cloud.cnaf.infn.it/
base_path = /data/grid/wlcg
map_subject = false
default_user = xrootd
```

- Note that the `onmissing = passthrough` part is needed to continue with
  other authorization libraries, such as the Macaroons library,
  and to continue with the evaluation of an authdb file (if used).

- Extend your existing xrootd configuration file. To stack with the
  macaroons authentication library, you will need:
```
ofs.authlib ++ libXrdAccSciTokens.so config=/etc/xrootd/scitokens.cfg
ofs.authlib ++ libXrdMacaroons.so 
ofs.authorize 1

# Pass the bearer token to the Xrootd authorization framework.
http.header2cgi Authorization authz
```
- The `++` is needed for stacking of the authorization libraries.
  If this is not needed, i.e. you only use one library,
  the `++` can be dropped.

- In case an `acc.authdb` file is used, authorization can be granted
  as follows, granting full read-write access to the `/data/grid/wlcg`
  path and read-only and list access to `/data/grid/srr`:
```
= wlcgtknusr o: https://wlcg.cloud.cnaf.infn.it/ g: /wlcg
x wlcgtknusr /data/grid/wlcg a  /data/grid/srr lr
```

This configuration enables flat authorization access on the storage to members
of the WLCG VO with the `/wlcg` group,, i.e. all users will have read and write
access to the data at `/data/grid/wlcg`.

- [WLCG JWT compliance testbed](https://github.com/indigo-iam/wlcg-jwt-compliance-tests#storage-area-configuration-pre-requisites)
  expect also `protected` resource accessible only with optional group
  `/wlcg/test` and this can be configured by following lines in
  the `acc.authdb` file for WLCG VO data at `/data/grid/wlcg`
  (it is also possible to specify `oss.localroot /data/grid`
  and than everything can be configured and used without this prefix)
```
# compound identity to support for group based authorization from WLCG JWT token
= wlcgtknprt_token o: https://wlcg.cloud.cnaf.infn.it/ g: /wlcg/test
= wlcgtknusr_token o: https://wlcg.cloud.cnaf.infn.it/ g: /wlcg
# compound identity to support for mapping X.509 VOMS identity
= wlcgtknprt_x509 o: wlcg g: /wlcg r: test
= wlcgtknusr_x509 o: wlcg g: /wlcg
# templates for accessing normal and protected resources
t wlcgtknprt /data/grid/wlcg a /data/grid lr
t wlcgtknusr /data/grid/wlcg/protected rl-diknw /data/grid/wlcg a /data/grid lr
# configure access for users that comes with X.509 or WLCG JWT token with wlcg.groups
# (with "x" first matching compound identity is used to grant privileges)
x wlcgtknprt_token wlcgtknprt
x wlcgtknusr_token wlcgtknusr
x wlcgtknprt_x509 wlcgtknprt
x wlcgtknusr_x509 wlcgtknusr
# WLCG JWT token scope based access is not handled in this configuration file
```
## Enabling tokens for ATLAS

### Simple native XRootD configuration

If you use very simple XRootD configuration with posix backend and `acc.authdb`
to specify permission for clients using X.509 VOMS proxy certificate than it
should be sufficient to add `/etc/xrootd/scitokens.cfg` configuration file
for `xrd.example.com:1094` and top level VO directory `/your/base/path/for/atlas`
```
# /etc/xrootd/scitokens.cfg
[Global]
onmissing = passthrough
audience = https://xrd.example.com:1094

[Issuer ATLAS]
issuer = https://atlas-auth.web.cern.ch/
base_path = /your/base/path/for/atlas
map_subject = False
default_user = xrootd
```
Plus update `ofs.authlib` in your XRootD configuration file with `libXrdAccSciTokens.so`
as mentioned in section with WLCG compliance testbed configuration.

ATLAS plans to rely exclusively on storage scopes in the tokens and that's why
your production `acc.authdb` configuration should not contain any mapping for
`wlcg.groups` that can be present in the token. All accesses to the storage with
tokens that doesn't contain relevant storage scopes should be rejected.

### EOS mapping with directories using different identity

EOS migrated from CASTOR and configured with full compatibility
with original CASTOR storage rely only on certificate subject
/ grid-mapfiles for identity mapping. This configuration could
be equally translated for clients that comes with tokens, e.g.

* /etc/xrootd/scitokens.cfg
```
# /etc/xrootd/scitokens.cfg
[Global]
onmissing = passthrough
audience = https://wlcg.cern.ch/jwt/v1/any, roots://eosatlas.cern.ch:1094, https://eosatlas.cern.ch, eosatlas.cern.ch

[Issuer ATLAS]
issuer = https://atlas-auth.web.cern.ch/
base_path = /eos/atlas
map_subject = False
name_mapfile = /etc/xrootd/scitokens.map
default_user = atlas001
```
* /etc/xrootd/scitokens.map (map all non-default users with different privileges)
```
[
  {"sub": "00000000-0000-0000-0000-000000000002", "result": "atlas002", "comment": "/DC=ch/DC=cern/OU=Organic Units/OU=Users/CN=user002/CN=000002/CN=Robot: ATLAS User 2"},
  {"sub": "00000000-0000-0000-0000-000000000003", "result": "atlas003", "comment": "/DC=ch/DC=cern/OU=Organic Units/OU=Users/CN=user003/CN=000003/CN=Robot: ATLAS User 3"},
  ...
]
```

This kind of mapping assumes that directory owners on EOS
match exactly storage scope path restriction defined in IAM.
May be it is even better to rely primarily on storage scope
path policies defined directly in IAM and than just map paths
to the identity that match owner it the EOS namespace, e.g.

```
[
  {"path": "/eos/atlas/atlasscratchdisk", "result": "atlas001", "comment": "Owner of the ATLASSCRATCHDISK area"},
  {"path": "/eos/atlas/atlasdatadisk", "result": "atlas003", "comment": "Owner of the ATLASDATADISK area"},
  ...
]
```


[xrootd]: https://xrootd.slac.stanford.edu/
[xrootd-scitokens]: https://github.com/xrootd/xrootd-scitokens
