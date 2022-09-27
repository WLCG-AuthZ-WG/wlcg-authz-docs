# Requirements

## WLCG Compliance testsuite storage configuration

Described in details in the WLCG JWT compliance testsuite [README](https://github.com/indigo-iam/wlcg-jwt-compliance-tests/blob/master/README.md)

## Alice Storage

Already use their own proprietary tokens (not WLCG JWT tokens) and that means everything is mapped to one storage identity, because their security model already rely on capabilities.

## ATLAS Storage

Rucio file replication with FTS always use production role when writing files in the "rucio" subdirectory
and while deleting files. For jobs using rucio upload the identity used while writing files differs
for production (`FQAN:/atlas/Role=production` identity) and analysis jobs (`FQAN:/atlas` identity). Also user
can store own files with rucio upload and normal `FQAN:/atlas` identity.

Usual namespace organization with access permission:

* `/basepath/atlas` - read only for `FQAN:/atlas` group
* `/basepath/atlas/atlasscratchdisk` - inheritable read for `FQAN:/atlas` and write for `FQAN:/atlas` + `FQAN:/atlas/Role=production`
* `/basepath/atlas/atlasdatadisk` - inheritable read for `FQAN:/atlas` and write for role `FQAN:/atlas/Role=production`
* `/basepath/atlas/atlaslocalgroupdisk` - inheritable read for `FQAN:/atlas` and write for group `FQAN:/atlas/country_code` + `FQAN:/atlas/Role=production`

All files can be read with basic `FQAN:/atlas` VO identity.

Rely exclusively on capabilities that comes with token `storage.*` scopes and completely ignore `wlcg.groups` in the token for storage access.

## CMS Storage

* `/basepath/cms` - everything readable by `FQAN:/cms` identity
* `/basepath/cms/{data,mc,...}` - write for role `FQAN:/cms/Role=production`
* `/basepath/cms/group` - write for role `FQAN:/cms/Role=priorityuser`
* `/basepath/cms/group/rucio` - write for role `FQAN:/cms/Role=production`
* `/basepath/cms/user` - only for local users from same home institute (may even provide per-user access permissions)

Rely exclusively on capabilities that comes with token `storage.*` scopes and completely ignore `wlcg.groups` in the token for storage access.

## LHCb Storage

Distinguish two VOMS roles: user and production.

* `/basepath/lhcb` - read/write everywhere with production role `FQAN:/lhcb/Role=production`, everything readable by `FQAN:/lhcb`
* `/basepath/lhcb/user` - the only directory writeable by normal users with just basic `FQAN:/lhcb`

Prefer capabilities that comes with token `storage.*` scopes (depends on Dirac development and may still consider `wlcg.groups` when it turns out it is too difficult to use capabilities within Dirac framework).
