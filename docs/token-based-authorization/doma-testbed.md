# DOMA Token-based AuthZ Testbed 

## Joining testbed

Send mail to [wlcg-doma-tpc e-group](https://e-groups.cern.ch/e-groups/Egroup.do?egroupId=10301575) if you want to join [Rucio DOMA Functional Tests OIDC](https://atlas-kibana.mwt2.org:5601/s/rucio/goto/07442ef87b3ba72bc391edde8770de4b) or advertise your CE that support job submission with tokens. Please include necessary to access your SE/CE endpoint, as an example you can use entries that were already filled in tables with available resources that to some degree already supports tokens.

## IAM

 VO  | Issuer
---- | ------
WLCG IAM | https://wlcg.cloud.cnaf.infn.it/
ATLAS IAM | https://atlas-auth.web.cern.ch/
CMS IAM | https://cms-auth.web.cern.ch/
DUNE | https://cilogon.org/dune
Fermilab | https://cilogon.org/fermilab

## CE

### [ARC-CE](https://www.nordugrid.org/arc/arc6/misc/oidc_tokens.html)

ARC-CE 6.12 still has limited support for WLCG JWT tokens and they can be used
submit jobs with HTTP based protocols (`EMI-ES` and `REST` interface). With current
limitations configuration that is close to expectation from WLCG JWT profile
for WLCG IAM could look like

```
# ...
[authtokens]

[authgroup: wlcg_iam]
authtokens = * https://wlcg.cloud.cnaf.infn.it/ * compute.create /wlcg/pilots
authtokens = * https://wlcg.cloud.cnaf.infn.it/ * compute.read /wlcg/pilots
authtokens = * https://wlcg.cloud.cnaf.infn.it/ * compute.modify /wlcg/pilots
authtokens = * https://wlcg.cloud.cnaf.infn.it/ * compute.cancel /wlcg/pilots

# just an example for ARC-CE running on arc1.farm.particle.cz
# recommendation for ATLAS configuration may change in fugure
# (this is not the official ATLAS site configuration documentation)
[authgroup: iam_atlas_group]
authtokens = * https://atlas-auth.web.cern.ch/ https://arc1.farm.particle.cz compute.create *
authtokens = * https://atlas-auth.web.cern.ch/ https://arc1.farm.particle.cz compute.read *
authtokens = * https://atlas-auth.web.cern.ch/ https://arc1.farm.particle.cz compute.modify *
authtokens = * https://atlas-auth.web.cern.ch/ https://arc1.farm.particle.cz compute.cancel *

# again, just an example for ARC-CE running on arc1.farm.particle.cz
# (this is not the official CMS site configuration documentation)
[authgroup: iam_cms_group]
authtokens = * https://cms-auth.web.cern.ch/ https://arc1.farm.particle.cz compute.create /cms/pilot
authtokens = * https://cms-auth.web.cern.ch/ https://arc1.farm.particle.cz compute.read /cms/pilot
authtokens = * https://cms-auth.web.cern.ch/ https://arc1.farm.particle.cz compute.modify /cms/pilot
authtokens = * https://cms-auth.web.cern.ch/ https://arc1.farm.particle.cz compute.cancel /cms/pilot

# this assumes existence of local users wlcg, atlas and cms with corresponding groups
[mapping]
map_to_user = wlcg_iam wlcg:wlcg
map_to_user = iam_atlas_group atlas:atlas
map_to_user = iam_cms_group cms:cms
policy_on_nomap=stop

[arex/ws/jobs]
allowaccess=wlcg_iam
allowaccess=atlas_iam
allowaccess=cms_iam
# ...
```

Token implementation in ARC-CE is still preliminary and it seems possible there
will be changes even in the structure of configuration file. You should [follow
official documentation](https://www.nordugrid.org/arc/arc6/misc/oidc_tokens.html)
if you install more recent ARC-CE version.

Even though job submission is possible with tokens it is still necessary to have
existing X.509 proxy, because current `arcsub` version still unconditionally verify
proxy presence. On the other hand this proxy is delegated to ARC-CE regardless
of auth mechanism used to submit jobs.

With token in `BEARER_TOKEN` environement variable (`ARC_OTOKEN` for older ARC-CE 6.7
clients) token'll be automatically used for authorization with HTTP interface, e.g.
```
cat > test.xrsl <<EOF
&(executable="/usr/bin/env")
(stdout="test.out")
(stderr="test.err")
(jobname="ARC-CE test")
(runtimeenvironment = "ENV/PROXY")
EOF
# following line is necessary for ARC-CE 6.7 client
# newer version use BEARER_TOKEN or token discovery
#export ARC_OTOKEN=$(oidc-token name)
export BEARER_TOKEN=$(oidc-token name)
arcsub --debug=DEBUG --info-endpoint-type arcrest --submission-endpoint-type arcrest --computing-element arc1.farm.particle.cz test.xrsl
```
ARC-CE support token discovery as described in [WLCG Bearer Token Discovery](https://zenodo.org/record/3937438)

### Configured endpoints
Site | Host | VO | Audience | Group
---- | ---- | -- | -------- | -----
[praguelcg2](https://goc.egi.eu/portal/index.php?Page_Type=Site&id=288) | arc1.farm.particle.cz, arc2.farm.particle.cz | wlcg | * | /wlcg/pilots
[praguelcg2](https://goc.egi.eu/portal/index.php?Page_Type=Site&id=288) | arc1.farm.particle.cz, arc2.farm.particle.cz | atlas, dune | * | *


### [HTCondor-CE](https://htcondor.github.io/htcondor-ce/v5/installation/htcondor-ce/)

It is possible to submit jobs with WLCG JWT token if HTCondor accepts `SCITOKENS`.
By default HTCondor-CE is configured with [non-standard `aud` claim](https://github.com/WLCG-AuthZ-WG/common-jwt-profile/blob/master/profile.md#common-claims), but this can
be changed by adding following line in `/etc/condor-ce/config.d/99-local` file
```
SCITOKENS_SERVER_AUDIENCE = $(SCITOKENS_SERVER_AUDIENCE) condor://$(COLLECTOR_HOST)
# special "any" audience, not recommended for production use
#SCITOKENS_SERVER_AUDIENCE = $(SCITOKENS_SERVER_AUDIENCE) https://wlcg.cern.ch/jwt/v1/any
```
All other configuration and identity mapping is described in the
[documentation](https://htcondor.github.io/htcondor-ce/v5/configuration/authentication/).
Currently identity mapping is limited to `sub`+`iss` which is not sufficient for IAM shared
by multiple groups (e.g. Fermialab), but there are plans to implement callout similar
to the GSI lcmaps.

Follow these steps to submit job with token to HTCondor-CE
```
cat > test.sub <<EOF
executable = /bin/env
output = env.out
error = env.err
log = env.log
queue
EOF

# get access token, e.g. from oidc-agent (see bellow) and store its
# content in a file used by WLCG bearer token discovery mechanism
oidc-token --aud=condor://osgce3.farm.particle.cz:9619 -s compute.create -s compute.read -s compute.modify -s compute.cancel wlcg-compute > $XDG_RUNTIME_DIR/bt_u$(id -u)

export CONDOR_CONFIG=/dev/null
export GSI_AUTHZ_CONF=/dev/null
export _condor_AUTH_SSL_CLIENT_CADIR=/etc/grid-security/certificates
export _condor_SEC_CLIENT_AUTHENTICATION_METHODS=SCITOKENS

condor_ping -verbose -pool osgce3.farm.particle.cz:9619 -name osgce3.farm.particle.cz WRITE
condor_submit -pool osgce3.farm.particle.cz:9619 -remote osgce3.farm.particle.cz test.sub
condor_q -pool osgce3.farm.particle.cz:9619 -name osgce3.farm.particle.cz
```

#### Obtain access token with compute.* scopes

* oidc-agent
```
# start oidc-agent (if it is not already running)
eval $(oidc-gen)
# register new client in oidc-agent with compute.* scopes (one time
# action, but not all IAM allows user to apply for compute.* scopes)
oidc-gen --iss=https://wlcg.cloud.cnaf.infn.it/ --scope="openid offline_access compute.create compute.read compute.modify compute.cancel" wlcg-compute
# obtain token with all necessary scopes and right audience
oidc-token --aud=condor://osgce3.farm.particle.cz:9619 --scope "compute.create compute.read compute.modify compute.cancel" wlcg-compute > $XDG_RUNTIME_DIR/bt_u$(id -u)
```
* client credentials grant (client registered in IAM with `compute.*` scopes, grant type "client credentials" and response type "token")
```
curl -s --data "grant_type=client_credentials&client_id=client_id_from_iam_registration&client_secret=client_secret_from_iam_registration&audience=condor://osgce3.farm.particle.cz:9619" https://wlcg.cloud.cnaf.infn.it/token | jq -r .access_token > $XDG_RUNTIME_DIR/bt_u$(id -u)
```

### Configured endpoints
Site | Host | VO | Audience
---- | ---- | -- | --------
osg | hosted-ce-chtc-ubuntu.osg.chtc.io:9619 | wlcg | hosted-ce-chtc-ubuntu.osg.chtc.io:9619
[praguelcg2](https://goc.egi.eu/portal/index.php?Page_Type=Site&id=288) | osgce3.farm.particle.cz:9619 | wlcg, atlas, dune | osgce3.farm.particle.cz:9619, condor://osgce3.farm.particle.cz:9619, https://wlcg.cern.ch/jwt/v1/any

## Storage

### Implementations
* [dCache](https://www.dcache.org/manuals/Book-6.2/config-gplazma.shtml#using-openid-connect) ([transfer documentation](https://www.dcache.org/manuals/UserGuide-6.2/webdav.shtml#authorising-the-data-transfer))
* [DPM](https://twiki.cern.ch/twiki/bin/view/DPM/DpmSetupPuppetInstallation#OpenID_Connect_and_WLCG_bearer_t)
* Echo (CEPH + XRootD plugin)
* EOS
* StoRM ([configuration instructions](./configuration/storm.md))
* XRootD

### Configured endpoints:
Site | Implementation | Host | VO | Audience
---- | -------------- | ---- | -- | --------
[praguelcg2](https://goc.egi.eu/portal/index.php?Page_Type=Site&id=288) | DPM 1.15.0 | https://golias100.farm.partile.cz/dpm/farm.particle.cz/home/wlcg | wlcg | https://wlcg.cern.ch/jwt/v1/any
UNL | XRootD | https://red-gridftp12.unl.edu:1094/user/dteam | wlcg |
DESY Devel | dCache 7.1 | https://prometheus.desy.de:2443/VOs/wlcg | wlcg |
CNAF Prod | StoRM | https://xfer.cr.cnaf.infn.it:8443/wlcg | wlcg |
CNAF Devel | StoRM | https://amnesiac.cloud.cnaf.infn.it:8443/wlcg | wlcg |
CERN ATLAS | EOS | https://eosatlas.cern.ch:443/eos/atlas/atlasscratchdisk/3rdpartycopy | atlas |
CERN Devel | EOS | https://eospps.cern.ch:443/eos/opstest/tpc/https | wlcg |
RAL | Echo | https://ceph-gw8.gridpp.rl.ac.uk:1094/dteam:test/ | wlcg |
Manchester Test | DPM | https://vm33.in.tier2.hep.manchester.ac.uk:443/dpm/tier2.hep.manchester.ac.uk/home/wlcg | wlcg |

## [FTS](http://fts3-docs.web.cern.ch/fts3-docs/docs/install/fts3.html)

[CERN Devel FTS](https://fts3-devel.cern.ch:8449) with 3.10.x provides JWT support for WLCG and XDC.
```
fts-rest-whoami --access-token=<token> -s https://fts3-devel.cern.ch:8446
fts-rest-transfer-submit --access-token=<token> -s https://fts3-devel.cern.ch:8446 <src_url> <dst_url>
fts-rest-transfer-status --access-token=<token> -s https://fts3-devel.cern.ch:8446
```
Be aware that for sucessfull FTS transfer submission with OIDC you also need recent 3.10.x FTS rest client.

## RUCIO

Install Rucio client with one method described in [documentation](https://rucio.readthedocs.io/en/latest/installing_clients.html), e.g.
```shell
# latest rucio client works only with python3
virtualenv-3 rucio
source rucio/bin/activate
pip3 install rucio-clients
pip3 install gfal2-python
```
and save following configuration file in `rucio/etc/rucio.cfg` for WLCG DOMA Rucio OIDC tests
```shell
[client]
rucio_host = https://rucio-doma.cern.ch:443
auth_host = https://rucio-doma-auth.cern.ch:443
auth_type = oidc
account = wlcg_doma
oidc_issuer = wlcg
ca_cert = /etc/grid-security/certificates
#ca_cert = /etc/pki/tls/certs/CERN-bundle.pem
```
Setup environment for Rucio client installed in virtualenv
```shell
source rucio/bin/activate
```
or if you use different installation method just set `RUCIO_HOME` environment
variable to the base directory with `etc/rucio.cfg` file
```shell
export RUCIO_HOME=/base/path/to/rucio
```

[WLCG IAM account](https://wlcg.cloud.cnaf.infn.it) is necessary to
access [WLCG DOMA Rucio instance](https://rucio-doma-webui.cern.ch)
and user sub claim (WLCG IAM uuid identity) must be associated with
`wlcg_doma` Rucio account by DOMA Rucio ADMIN. It is also possible to
associate user certificate subject with `wlcg_doma` Rucio account to
provide access with WLCG VO X.509 certificate proxy, but for different
authorization type it is necessary to update `auth_type = x509_proxy`
in `rucio.cfg` or setting environment variable
`RUCIO_AUTH_TYPE=x509_proxy`.

## Monitoring

- [Functional Tests with WLCG OIDC](https://atlas-kibana.mwt2.org:5601/s/rucio/app/kibana#/dashboard/58b37580-b601-11ea-a57f-cfdaa2417d86)
- [Rucio DOMA Configuration](https://gitlab.cern.ch/wlcg-doma-rucio/flux/-/tree/master/releases/integration)
- [Rucio DOMA Logs](https://atlas-kibana.mwt2.org:5601/s/rucio/goto/2cdbb5f83cbe74358ed4287a764426dd)
- [CERN FTS3 Devel](https://fts3-devel.cern.ch:8449/fts3/ftsmon/#/)
- [FTS Grafana](https://monit-grafana.cern.ch/d/000000670/fts-servers-dashboard?orgId=25&var-group_by=activity&var-vo=dteam&var-vo_es=dteam&var-source_se=All&var-source_se_es=All&var-src_country=All&var-dst_country=All&var-src_site=All&var-dst_site=All&var-dest_se=All&var-dest_se_es=All&var-fts_server=fts3-devel.cern.ch&var-fts_server_es=fts3-devel.cern.ch&var-bin=1h&var-dest_hostname=All&var-source_hostname=All)
