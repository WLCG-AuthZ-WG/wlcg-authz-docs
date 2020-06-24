# StoRM 

Token-based authentication and authorization is supported by the latest [StoRM WebDAV][storm-webdav] release.

## Enabling token-based authN/Z for the WLCG IAM instance

In order to enable token-based AuthN/Z for the WLCG IAM instance on a StoRM
WebDAV deployment, you need to:

- Add the WLCG issuer to the list of the trusted issuers by the StoRM WebDAV
  instance. This is done by modifying the
  `/etc/storm/webdav/config/application.yml` as follows:

```
oauth:
  issuers:
  - name: iam-wlcg
    issuer: https://wlcg.cloud.cnaf.infn.it/
```

- Enable the issuer for wlcg the storage area, by changing the `wlcg` storage
  area configuration `/etc/storm/webdav/sa.d/wlcg.properties` like in the
  following example:

```
name=wlcg
rootPath=/your/storage/path/
accessPoints=/wlcg
vos=wlcg
orgs=https://wlcg.cloud.cnaf.infn.it/
authenticatedReadEnabled=false
anonymousReadEnabled=false
```

This configuration enables flat authorization access on the storage to members
of the WLCG VO, i.e. all users will have read and write access to the data.

## HTTP TPC support configuration

To configure support for HTTP TPC, follow the instructions in the [StoRM WebDAV
documentation][storm-webdav-tpc-doc].

[storm-webdav]: http://italiangrid.github.io/storm/release-notes/storm-webdav/1.2.1/
[storm-webdav-tpc-doc]: https://github.com/italiangrid/storm-webdav/blob/master/doc/tpc.md

