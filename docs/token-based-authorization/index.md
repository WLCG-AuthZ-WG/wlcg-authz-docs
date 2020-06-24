# Token-based authorization

## Getting an account on the `wlcg` IAM instance

To register in the WLCG IAM instance, point your browser to the `wlcg` 
[IAM VO page](https://wlcg.cloud.cnaf.infn.it).

If you have a CERN account, use that account to login to the `wlcg` VO.

## Registering a client in the WLCG IAM instance

The recommended way of registering a client in the WLCG IAM instance is to use
`oidc-agent` as described [here](./oidc-agent.md).

By default, clients registered in the WLCG IAM instance will use the [common
WLCG JWT profile][wlcg-jwt-profile].

## WLCG IAM scope policy configuration

IAM allows to define policies to limit access to scopes only to selected
classes of users.

In the IAM `wlcg` instance policies are defined to limit access to `compute.*`
and `storage.*` scopes, which are the scopes defined in the [WLCG JWT
profile][wlcg-jwt-profile] to control access to compute and storage resources.

In more detail:

- access to the `compute.create`, `compute.read`, `compute.cancel`,
  `compute.modify` scope is only granted to members of the `wlcg/pilots` group;

- access to the `storage.read`, `storage.modify`, `storage.create` scopes is
  only granted to members of the `wlcg/xfers` group.

Membership in those groups can be requested, after registration, from the IAM
dashboard.

## Getting tokens out IAM

### Scope-based authorization

In order to transfer data across WLCG storage elements using scope-based
authorization, request a token with the 'storage.read:/' and 'storage.modify:/'
scopes, as follows:

```
$ AT=$(oidc-token -s storage.read:/ -s storage.modify:/ wlcg)
```

You can use the `jwt` utility to inspect the token contents:


```
$ echo $AT | jwt
...
✻ Payload
{
  "wlcg.ver": "1.0",
  "sub": "a1b98335-9649-4fb0-961d-5a49ce108d49",
  "aud": "https://wlcg.cern.ch/jwt/v1/any",
  "nbf": 1593004542,
  "scope": "storage.read:/ storage.modify:/",
  "iss": "https://wlcg.cloud.cnaf.infn.it/",
  "exp": 1593008142,
  "iat": 1593004542,
  "jti": "da0a2f89-3cbf-42a7-9403-0b43d814551d",
  "client_id": "edfacfb1-f59d-44d0-9eb6-a745ac52f462"
}
```

Then you can use `davix` to access WLCG storage:

```
$ davix-ls -l --capath /etc/grid-security/certificates/ -H "Authorization: Bearer ${AT}" https://xfer.cr.cnaf.infn.it:8443/wlcg/

drwxrwxrwx 0     0          2020-06-24 10:41:13 wlcgdoma
-rwxrwxrwx 0     1048576    2020-06-18 11:16:25 1M
-rwxrwxrwx 0     1048576    2020-06-19 08:49:34 x
drwxrwxrwx 0     0          2020-04-08 14:56:56 https
drwxrwxrwx 0     0          2020-06-24 15:18:36 test-andrea
drwxrwxrwx 0     0          2020-06-22 13:06:24 vokac.fts-test

$ davix-ls -l --capath /etc/grid-security/certificates/ -H "Authorization: Bearer ${AT}" https://prometheus.desy.de:2443/VOs/wlcg

drwxrwxrwx 0     0          2020-06-24 06:01:59 Tape
drwxrwxrwx 0     0          2020-06-24 06:11:33 Private
-rwxrwxrwx 0     173        2020-06-24 06:11:33 public-file
-rwxrwxrwx 0     1048576    2020-06-24 14:14:30 x
drwxrwxrwx 0     0          2020-06-24 10:41:13 wlcgdoma
-rwxrwxrwx 0     150        2020-06-24 06:11:34 private-file

$ davix-ls -l --capath /etc/grid-security/certificates/ -H "Authorization: Bearer ${AT}" https://golias100.farm.particle.cz/dpm/farm.particle.cz/home/wlcg
-rw-rw-r-- 0     1048576    2020-06-18 08:12:23 1M
drwxrwx--- 0     1048576    2020-04-01 10:13:52 acl1
drwxrwxr-x 0     85983232   2020-06-19 22:19:00 doma
drwxrwxr-x 0     14680064   2020-06-22 13:06:25 vokac.fts-test
drwxrwxr-x 0     52428800   2020-06-24 10:41:13 wlcgdoma
-rw-rw-r-- 0     1048576    2020-06-24 13:01:03 x
-rw-rw-r-- 0     1048576    2020-03-31 00:20:00 x.pull
-rw-rw-r-- 0     1048576    2020-03-31 00:20:00 x.push
-rw-rw-r-- 0     1048576    2020-06-05 11:49:59 x1
```

### Group-based authorization

TBD

[wlcg-jwt-profile]: https://zenodo.org/record/3460258
