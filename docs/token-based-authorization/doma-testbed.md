# DOMA Token-based AuthZ Testbed 

## Endpoints

###  IAM

- https://wlcg.cloud.cnaf.infn.it

### RUCIO

Install [rucio client](https://rucio.readthedocs.io/en/latest/installing_clients.html)
```shell
virtualenv rucio
source rucio/bin/activate
pip install rucio-clients
mkdir rucio/etc
cat > rucio/etc/rucio.cfg <<EOF
[client]
rucio_host = https://rucio-doma.cern.ch:443
auth_host = https://rucio-doma-auth.cern.ch:443
auth_type = x509_proxy
# admin credentials
#auth_type = userpass
#username = ddmlab
#password = secret
account = root
ca_cert = /etc/grid-security/certificates
request_retries = 3
EOF
```
Setup rucio client
```shell
source rucio/bin/activate
export RUCIO_HOME=`pwd`/rucio
```

### FTS

[CERN Devel FTS](https://fts3-devel.cern.ch:8449) with 3.10.x provides JWT support for WLCG and XDC.
```
fts-rest-transfer-submit --access-token=<token> -s https://fts3-devel.cern.ch:8446 <src_url> <dst_url>
fts-rest-transfer-status --access-token=<token> -s https://fts3-devel.cern.ch:8446
```

### dCache

- [Prometheus dev endpoint](https://prometheus.desy.de:2443/VOs/wlcg)

### DPM

- [PRAGUE production endpoint](https://golias100.farm.particle.cz:443/dpm/farm.particle.cz/home/wlcg)

### Echo

- ?

### EOS

- [CERN dev endpoint](https://eospps.cern.ch:443/eos/opstest/tpc/https) (not yet ready / working)

### StoRM

-  [CNAF production endpoint](https://xfer.cr.cnaf.infn.it:8443/wlcg)
-  [CNAF dev endpoint](https://storm.cloud.cnaf.infn.it/wlcg) (not yet active)
  
### XRootD

- [Nebraska dev endpoint](https://red-gridftp12.unl.edu:1094/user/dteam)
