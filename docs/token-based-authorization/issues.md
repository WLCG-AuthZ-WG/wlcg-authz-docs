# Known issues

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

## EOS

## RUCIO

## FTS
