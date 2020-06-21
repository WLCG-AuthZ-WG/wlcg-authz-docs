# DOMA Token-based AuthZ Testbed 

## Endpoints

###  IAM

- https://wlcg.cloud.cnaf.infn.it

### RUCIO

Install Rucio client with one method described in [documentation](https://rucio.readthedocs.io/en/latest/installing_clients.html), e.g.
```shell
virtualenv rucio
source rucio/bin/activate
pip install rucio-clients
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
or if you use different installation method just set `RUCIO_HOME` environment variable to the base directory with `etc/rucio.cfg` file
```shell
export RUCIO_HOME=/path/to/your/rucio
```
WLCG IAM account is necessary to access WLCG DOMA Rucio instance user sub claim (WLCG IAM uuid identity) must be associated with `wlcg_doma` Rucio account.

### [FTS](http://fts3-docs.web.cern.ch/fts3-docs/docs/install/fts3.html)

[CERN Devel FTS](https://fts3-devel.cern.ch:8449) with 3.10.x provides JWT support for WLCG and XDC.
```
fts-rest-transfer-submit --access-token=<token> -s https://fts3-devel.cern.ch:8446 <src_url> <dst_url>
fts-rest-transfer-status --access-token=<token> -s https://fts3-devel.cern.ch:8446
```
Be aware that for sucessfull FTS transfer submission with OIDC you also need recent 3.10.x FTS rest client.

### [dCache](https://www.dcache.org/manuals/Book-6.2/config-gplazma.shtml#using-openid-connect) ([https://www.dcache.org/manuals/UserGuide-6.2/webdav.shtml#authorising-the-data-transfer](transfer documentation) )

- [Prometheus dev endpoint](https://prometheus.desy.de:2443/VOs/wlcg)

### [DPM](https://twiki.cern.ch/twiki/bin/view/DPM/DpmSetupPuppetInstallation#OpenID_Connect_and_WLCG_bearer_t)

- [PRAGUE production endpoint](https://golias100.farm.particle.cz:443/dpm/farm.particle.cz/home/wlcg)

### Echo

- ?

### EOS

- [CERN dev endpoint](https://eospps.cern.ch:443/eos/opstest/tpc/https) (not yet ready / working)
- [CERN ATLAS production endpoint](https://eosatlas.cern.ch:443/eos/atlas/atlasscratchdisk/3rdpartycopy) (only for ATLAS members)

### StoRM

- [CNAF production endpoint](https://xfer.cr.cnaf.infn.it:8443/wlcg)
- [CNAF dev endpoint](https://storm.cloud.cnaf.infn.it/wlcg) (not yet active)

### XRootD

- [Nebraska dev endpoint](https://red-gridftp12.unl.edu:1094/user/dteam)