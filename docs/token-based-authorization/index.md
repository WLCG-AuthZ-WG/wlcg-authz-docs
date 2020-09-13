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

or directly decode token payload with `echo $AT | sed 's/.*\.\(.*\)\..*/\1==/' | base64 -d | jq`

Then you can use `davix` or `curl` to access WLCG storage:

```
$ davix-ls -l --capath /etc/grid-security/certificates -H "Authorization: Bearer ${AT}" https://xfer.cr.cnaf.infn.it:8443/wlcg/
drwxrwxrwx 0     0          2020-06-24 10:41:13 wlcgdoma
-rwxrwxrwx 0     1048576    2020-06-18 11:16:25 1M
drwxrwxrwx 0     0          2020-04-08 14:56:56 https
drwxrwxrwx 0     0          2020-06-24 15:18:36 test-andrea

$ davix-put --capath /etc/grid-security/certificates -H "Authorization: Bearer ${AT}" /etc/services https://prometheus.desy.de:2443/VOs/wlcg/file1
$ davix-get --capath /etc/grid-security/certificates https://prometheus.desy.de:2443/VOs/wlcg/file1 /tmp/x
Performing Read operation on: http://[2001:638:700:1005:0:0:1:95]:21881/VOs/wlcg/file1?dcache-http-uuid=679aa9a4-b946-49fd-8522-156d165425c5
[==================================] 100%     654KiB/654KiB         0B/s
$ davix-mkdir --capath /etc/grid-security/certificates -H "Authorization: Bearer ${AT}" https://golias100.farm.particle.cz/dpm/farm.particle.cz/home/wlcg/dir1
$ davix-cp --capath /etc/grid-security/certificates -H "Authorization: Bearer ${AT}" -H "TransferHeaderAuthorization: Bearer ${AT}" --copy-mode pull https://prometheus.desy.de:2443/VOs/wlcg/file1 https://golias100.farm.particle.cz/dpm/farm.particle.cz/home/wlcg/dir1/file1.pull
$ davix-cp --capath /etc/grid-security/certificates -H "Authorization: Bearer ${AT}" -H "TransferHeaderAuthorization: Bearer ${AT}" --copy-mode pull https://prometheus.desy.de:2443/VOs/wlcg/file1 https://golias100.farm.particle.cz/dpm/farm.particle.cz/home/wlcg/dir1/file1.push
$ davix-rm --capath /etc/grid-security/certificates -H "Authorization: Bearer ${AT}" https://golias100.farm.particle.cz/dpm/farm.particle.cz/home/wlcg/dir1

# Create directory
$ curl --capath /etc/grid-security/certificates -X MKCOL -H "Authorization: Bearer ${AT}" https://golias100.farm.particle.cz/dpm/farm.particle.cz/home/wlcg/dir1
# Upload file
$ curl --capath /etc/grid-security/certificates -L -X PUT -H "Authorization: Bearer ${AT}" --upload-file /etc/services https://golias100.farm.particle.cz/dpm/farm.particle.cz/home/wlcg/dir1/file1
# Download file
$ curl --capath /etc/grid-security/certificates -L -X GET -H "Authorization: Bearer ${AT}" --output /tmp/x https://golias100.farm.particle.cz/dpm/farm.particle.cz/home/wlcg/dir1/file1
# TPC pull
$ curl --capath /etc/grid-security/certificates -L -X COPY -H 'Credentials: none' -H "Authorization: Bearer ${AT}" -H "TransferHeaderAuthorization: Bearer ${AT}" -H "Source: https://golias100.farm.particle.cz/dpm/farm.particle.cz/home/wlcg/dir1/file1" https://golias100.farm.particle.cz/dpm/farm.particle.cz/home/wlcg/dir1/file2
# TPC push
$ curl --capath /etc/grid-security/certificates -L -X COPY -H 'Credentials: none' -H "Authorization: Bearer ${AT}" -H "TransferHeaderAuthorization: Bearer ${AT}" -H "Destination: https://golias100.farm.particle.cz/dpm/farm.particle.cz/home/wlcg/dir1/file3" https://golias100.farm.particle.cz/dpm/farm.particle.cz/home/wlcg/dir1/file1
# Delete
$ curl --capath /etc/grid-security/certificates -L -X DELETE -H "Authorization: Bearer ${AT}" https://golias100.farm.particle.cz/dpm/farm.particle.cz/home/wlcg/dir1
```

Gfal python API can be also used with tokens, e.g.

```
import os
import gfal2
ctx = gfal2.creat_context()
cred = gfal2.cred_new("BEARER", os.getenv('TOKEN'))
gfal2.cred_set(ctx, 'amnesiac.cloud.cnaf.infn.it', cred)
ctx.stat('https://amnesiac.cloud.cnaf.infn.it:8443/wlcg/wlcgdoma/https/jwttest/1M')
```

### Group-based authorization

TBD

[wlcg-jwt-profile]: https://zenodo.org/record/3460258
