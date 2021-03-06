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
  (assuming `xrootd` should act as the `xrootd` user):
```
[Global]
onmissing = passthrough

[Issuer WLCG IAM]
issuer = https://wlcg.cloud.cnaf.infn.it/
base_path = /
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

[xrootd]: https://xrootd.slac.stanford.edu/
[xrootd-scitokens]: https://github.com/xrootd/xrootd-scitokens
