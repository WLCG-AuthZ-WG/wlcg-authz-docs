# Requirements

## WLCG Compliance testsuite storage configuration

Described in details in the WLCG JWT compliance testsuite [README](https://github.com/indigo-iam/wlcg-jwt-compliance-tests/blob/master/README.md)

## Alice Storage

Already use their own proprietary tokens (not these WLCG JWT tokens) and that means everything is mapped to one storage identity, because their security model already rely on capabilities.

## ATLAS Storage

Rucio file replication with FTS always use production role when writing files in the "rucio" subdirectory
and while deleting files. For jobs using rucio upload the identity used while writing files differs
for production (`/atlas/Role=production` identity) and analysis jobs (`/atlas` identity). Also user
can store own files with rucio upload and normal `/atlas` identity.

Usual namespace organization with access permission:

* `/basepath` - read only for `/atlas` group
* `/basepath/atlasscratchdisk` - inheritable read for `/atlas` and write for `/atlas` + `/atlas/Role=production`
* `/basepath/atlasdatadisk` - inheritable read for `/atlas` and write for role `/atlas/Role=production`
* `/basepath/atlaslocalgroupdisk` - inheritable read for `/atlas` and write for group `/atlas/country_code` + `/atlas/Role=production`

All files can be read with basic `/atlas` VO identity.

## CMS Storage

* `/basepath` - everything readable by `/cms` identity
* `/basepath/{data,mc,...}` - write for role `/cms/Role=production`
* `/basepath/group` - write for role `/cms/Role=priorityuser`
* `/basepath/group/rucio` - write for role `/cms/Role=production`
* `/basepath/user` - only for local users from same home institute (may even provide per-user access permissions)

## LHCb Storage

Distinguish two VOMS roles: user and production.

* `production` - read/write everywhere
* `user` - read everything but write only in the basepath for user
