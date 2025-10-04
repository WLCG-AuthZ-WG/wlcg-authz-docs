# StoRM 

Token-based authentication and authorization is supported by the StoRM WebDAV [version 1.2.1][storm-webdav].

## Enabling token-based authN/Z for the WLCG IAM instance

The following StoRM WebDAV configuration is based on the requirements for the [WLCG JWT compliance testsuite](https://github.com/indigo-iam/wlcg-jwt-compliance-tests#storage-area-configuration-pre-requisites).

In order to enable the token-based AuthN/Z for the WLCG IAM instance on a StoRM WebDAV deployment, you need to:

- Add the following properties for the wlcg storage area in the `/etc/storm/webdav/sa.d/wlcg.properties` configuration file:

```bash
# Name of the storage area
name=wlcg

# Root path for the storage area. Files will be served from this path, which must exist and
# must be accessible from the user that runs the storm webdav service
rootPath=/storage/wlcg

# Comma-separated list of storage area access points
accessPoints=/wlcg

# Comma-separated list of VOMS VOs supported in this storage area
vos=wlcg

# Comma-separated list of OAuth/OpenID Connect token issuers trusted in this storage area. 
# Such organizations must be included also in the list of trusted
# issuers specified in /etc/storm/webdav/config/application.yml
orgs=https://wlcg.cloud.cnaf.infn.it/

# Enables read access to storage area files to users authenticated using OAuth/OIDC. Defaults to true
orgsGrantReadPermission=false

# Enables scope-based authorization following the rules imposed by the WLCG JWT profile. Defaults to false
wlcgScopeAuthzEnabled=true

# Enables fine-grained authorization engine. Defaults to false
fineGrainedAuthzEnabled=true
```

- Add the following properties to the common
  `/etc/storm/webdav/config/application.yml` configuration file:

```yml
# Comma-separated list of OAuth/OpenID Connect token issuers trusted in this storage area. 
# Such issuers must be included also in the list of trusted
# organizations specified in /etc/storm/webdav/sa.d/wlcg.properties
oauth:
  issuers:
  - name: iam-wlcg
    issuer: https://wlcg.cloud.cnaf.infn.it/

# Check the audience claim as specified in the WLCG Common JWT Profiles
    enforce-audience-checks: true
    audiences:
        - https://wlcg.cern.ch/jwt/v1/any
        - https://<webdav-hostname>:<webdav-port>

# List of authorization policies which are applied when the 'fineGrainedAuthzEnabled' 
# property is set to true in /etc/storm/webdav/sa.d/wlcg.properties.
# The first policy that matches an incoming request will be applied
storm:
  authz:
    policies:

    - sa: wlcg
      description: Grant all access to /wlcg/protected to /wlcg/test members
      actions:
        - all
      paths:
        - /protected/**
      effect: permit
      principals:
        - type: fqan
          params:
            fqan: /wlcg/Role=test
        - type: jwt-group
          params:
            iss: https://wlcg.cloud.cnaf.infn.it/
            group: /wlcg/test

    - sa: wlcg
      description: Allow read access to /wlcg/protected area to wlcg members
      actions:
        - read
        - list
      paths:
        - /protected/**
      effect: permit
      principals:
        - type: vo
          params:
            vo: wlcg
        - type: jwt-group
          params:
            iss: https://wlcg.cloud.cnaf.infn.it/
            group: /wlcg

    - sa: wlcg
      description: Deny write access to /wlcg/protected area
      actions:
        - write
        - delete
      paths:
        - /protected/**
      effect: deny
      principals:
        - type: anyone

    - sa: wlcg
      description: Grant all access to the SA to wlcg members
      actions:
        - all
      effect: permit
      principals:
        - type: vo
          params:
            vo: wlcg
        - type: jwt-group
          params:
            iss: https://wlcg.cloud.cnaf.infn.it/
            group: /wlcg
```

This configuration enables a fine grained authorization access on the storage to members
of the WLCG VO, *i.e.* users presenting a valid X.509 VOMS proxy and/or a WLCG JWT token issued by the WLCG IAM instance.

The list of policies described by the `storm.authz.policies.description` property are applied to an authorization request when the `fineGrainedAuthzEnabled` property is set to true. Those rules follows a first-applicable principle, meaning that the first policy found in the list that matches an incoming request will be applied when rendering an authorization decision.
Further policies can be added following the StoRM WebDAV  [documentation](http://italiangrid.github.io/storm/documentation/sysadmin-guide/1.11.21/installation-guides/webdav/storage-area-configuration/index.html).

## HTTP TPC support configuration

To configure support for HTTP TPC, follow the instructions in the [StoRM WebDAV
documentation][storm-webdav-tpc-doc].

## VO Examples

### ATLAS

StoRM WebDAV configuration for the one and unique disk storage area “atlas” is
* /etc/storm/webdav/sa.d/atlas.properties
```
# The contents of this file was generated by Puppet
name=atlas
rootPath=/storage/gpfs_atlas/atlas
filesystemType=posix
accessPoints=/atlas
vos=
orgs=https://atlas-auth.cern.ch/
authenticatedReadEnabled=false
anonymousReadEnabled=false
voMapEnabled=false
voMapGrantsWritePermission=false
orgsGrantReadPermission=false
orgsGrantWritePermission=false
wlcgScopeAuthzEnabled=true
fineGrainedAuthzEnabled=true
```
* /etc/storm/webdav/config/application.yml
```yml
oauth:
  enable-oidc: true  
  issuers:
    - name: atlas
      issuer: https://atlas-auth.cern.ch/

storm:
  authz:
    policies:
    - sa: atlas
      description: Grant all access to atlas prod VOMS group members for /atlas/atlasdatadisk and its subfolders
      actions:
        - all
      paths:
        - /atlasdatadisk/**
      effect: permit
      principals:  
        - type: fqan
          params:
            fqan: /atlas/Role=production/Capability=NULL
    - sa: atlas
      description: Grant only read and list access to atlas VOMS group members for /atlas/atlasdatadisk and its subfolders
      actions:
        - read
        - list
      paths:
        - /atlasdatadisk/**
      effect: permit
      principals:
        - type: vo
          params:
            vo: atlas
    - sa: atlas
      description: Grant all access to atlas VOMS group members for /atlas/atlasscratchdisk and its subfolders
      actions:
        - all
      paths:
        - /atlasscratchdisk/**
      effect: permit
      principals:  
        - type: vo
          params:
            vo: atlas
    - sa: atlas
      description: Deny VO access to the access point
      actions:
        - all
      effect: deny
      principals:
        - type: vo
          params:
            vo: atlas
```

[storm-webdav]: http://italiangrid.github.io/storm/release-notes/storm-webdav/1.2.1/
[storm-webdav-tpc-doc]: https://github.com/italiangrid/storm-webdav/blob/master/doc/tpc.md

