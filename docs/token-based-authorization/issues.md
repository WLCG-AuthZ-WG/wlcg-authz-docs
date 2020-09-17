# Known issues

## oidc-agent

- [not available in EPEL repository](https://github.com/indigo-dc/oidc-agent/issues/267)

## Scitokens

- [storage.modify scope is not honoured by
  scitokens-cpp](https://github.com/scitokens/scitokens-cpp/issues/27):
  Small, but breaking issue since the scitokens library honours the
  `storage.write` instead of the `storage.modify` scope.
  Breaks WLCG JWT profile compatibility for any software that relies on the
  library.


## XRootD

## StoRM

- [StoRM WebDAV should drop Authorization header in TPC
  redirects](https://issues.infn.it/jira/browse/STOR-1197): this is solved and
  coming with the next StoRM release scheduled for the beginning of July;
- [StoRM WebDAV leaks file descriptors when Conscrypt is
  enabled](https://issues.infn.it/jira/projects/STOR/issues/STOR-1207): This is
  fixed in the code; a workaround to avoid this issue is disabling Conscrypt.
  This can be done by setting the following variables in
  `/etc/sysconfig/storm-webdav`:

```
STORM_WEBDAV_USE_CONSCRYPT="false"
STORM_WEBDAV_TPC_USE_CONSCRYPT="false"
STORM_WEBDAV_ENABLE_HTTP2="false"
```

## dCache

## DPM

- allows `HEAD` requests with just `scope.create:/`
- use `storage.write` scope not defined in WLCG JWT profile
- doesn't reject access tokens with expiration time longer than 6 hours
- files created with OIDC authentication has empty group owner

## EOS

## RUCIO

- token with `rucio` audience used for FTS file transfers (changed to `https://wlcg.cern.ch/jwt/v1/any` for Rucio DOMA instnace)
- default `scope` set to `openid profile` wich is not sufficient e.g. for `rucio upload` or `rucio download`
- fails in case there is expired X.509 proxy in `/tmp/x509up_u$(id -u)` although `auth_type` is set to `oidc`
- use [non-standard location for access token](https://github.com/WLCG-AuthZ-WG/bearer-token-discovery/blob/master/specification.md) `/tmp/$(id -u -n)/.rucio_${RUCIO_ACCOUNT}/auth_token_${RUCIO_ACCOUNT}`
- [CLI OIDC arguments differs from FTS REST client](https://gitlab.cern.ch/fts/fts-rest/-/merge_requests/34)

## FTS

## Gfal

- tokens can't be passed to `gfal-utils` CLI tools
- unable to determine which audience and scope `storage.*:/path` should be in used for SRM+HTTP transfers (TURL returned by SRM srmPrepareToGet + srmPrepareToPut not known before calling `copy` function)
