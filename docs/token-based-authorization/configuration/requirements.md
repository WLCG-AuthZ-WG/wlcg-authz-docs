# Requirements

## WLCG Compliance testsuite storage configuration

Described in details in the WLCG JWT compliance testsuite [README](https://github.com/indigo-iam/wlcg-jwt-compliance-tests/blob/master/README.md)

## Alice Storage

Already use their own proprietary tokens (not WLCG JWT tokens) and that means everything is mapped to one storage identity, because their security model already rely on capabilities.

## ATLAS Storage

Rucio file replication with FTS always use production role when writing files in the "rucio" subdirectory
and while deleting files. For jobs using rucio upload the identity used while writing files differs
for production (`/atlas/Role=production` identity) and analysis jobs (`/atlas` identity). Also user
can store own files with rucio upload and normal `/atlas` identity.

Usual namespace organization with access permission:

* `/basepath/atlas` - read only for `/atlas` group
* `/basepath/atlas/atlasscratchdisk` - inheritable read for `/atlas` and write for `/atlas` + `/atlas/Role=production`
* `/basepath/atlas/atlasdatadisk` - inheritable read for `/atlas` and write for role `/atlas/Role=production`
* `/basepath/atlas/atlaslocalgroupdisk` - inheritable read for `/atlas` and write for group `/atlas/country_code` + `/atlas/Role=production`

All files can be read with basic `/atlas` VO identity.

Rely exclusively on capabilities that comes with token `storage.*` scopes and completely ignore `wlcg.groups` in the token for storage access.

## CMS Storage

* `/basepath/cms` - everything readable by `/cms` identity
* `/basepath/cms/{data,mc,...}` - write for role `/cms/Role=production`
* `/basepath/cms/group` - write for role `/cms/Role=priorityuser`
* `/basepath/cms/group/rucio` - write for role `/cms/Role=production`
* `/basepath/cms/user` - only for local users from same home institute (may even provide per-user access permissions)

Rely exclusively on capabilities that comes with token `storage.*` scopes and completely ignore `wlcg.groups` in the token for storage access.

## LHCb Storage

Distinguish two VOMS roles: user and production.

* `/basepath/lhcb` - read/write everywhere with production role
* `/basepath/lhcb/user` - normal user read everything but write only in a specific subdirectory

Prefer capabilities that comes with token `storage.*` scopes (depends on Dirac development and may still consider `wlcg.groups` when it turns out it is too difficult to use capabilities within Dirac framework).
