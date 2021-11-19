# The WLCG Virtual Organization

An instance of IAM has been deployed in support of WLCG development and
integration activities in support of the migration to token-based
authentication and authorization.

The [WLCG IAM instance][wlcg-iam] is integrated with CERN SSO.

Registration is active. 

IAM documentation is available [here][iam-docs].

Clients applications can be registered following these
[instructions][client-registration], or using [oidc-agent][oidc-agent].

VOMS support is enabled. To link an X.509 certificate to an existing IAM WLCG
account, follow these [instructions][x509-linking]. As in VOMS, multiple
certificates can be linked to an account.

## WLCG VO VOMS configuration

!!! danger "New LSC files will be rolled out on Nov. 22nd, 2021"

      On Monday Nov. 22nd, 2021 at 11 we will update the WLCC VO LSC
      configuration. The reason for the update is that the current certificate
      is going to expire and that the Sectigo CA INFN currently uses has
      changed the structure of the DNs of the issued certificates. 

      The updated configuration that you find here is compliant with the new
      certificate. 

      **Do not roll out this configuration before Nov. 22nd, or VOMS validation
      will break at your site**.


- [VOMSES](./voms-config/wlcg-voms.cloud.cnaf.infn.it.vomses)
- [LSC](./voms-config/wlcg-voms.cloud.cnaf.infn.it.lsc)

To have a working VOMS configuration for the WLCG VO:

1. place the `lsc` file in the `/etc/grid-security/vomsdir/wlcg` directory 
2. place the `vomses` file in the `/etc/vomses` directory (only needed if you
   need to do `voms-proxy-init`)

### RPM installation

You can use the following RPM package to enable support for the WLCG VO on your
RHEL machine:

- [RPM
  package](https://repo.cloud.cnaf.infn.it/repository/infn-vos/centos7/infn-vo-wlcg-1.1.0-1.el7.noarch.rpm)

### VOMS clients compatibility

The latest supported VOMS clients are required.

Also note that this VO is supported by IAM, i.e. there are no VOMS Admin
endpoints that can be used to generate Gridmap files.

### Example

```console
$ voms-proxy-init -voms wlcg
Enter GRID pass phrase for this identity:
Contacting wlcg-voms.cloud.cnaf.infn.it:15001 [/DC=org/DC=terena/DC=tcs/C=IT/L=Frascati/O=Istituto Nazionale di Fisica Nucleare/CN=voms-wlcg.cloud.cnaf.infn.it] "wlcg"...
Remote VOMS server contacted succesfully.


Created proxy in /tmp/x509up_u501.

Your proxy is valid until Thu Jan 30 03:47:58 CET 2020

$ voms-proxy-info -all
subject   : /DC=org/DC=terena/DC=tcs/C=IT/O=Istituto Nazionale di Fisica Nucleare/CN=Andrea Ceccanti aceccant@infn.it/CN=743668640
issuer    : /DC=org/DC=terena/DC=tcs/C=IT/O=Istituto Nazionale di Fisica Nucleare/CN=Andrea Ceccanti aceccant@infn.it
identity  : /DC=org/DC=terena/DC=tcs/C=IT/O=Istituto Nazionale di Fisica Nucleare/CN=Andrea Ceccanti aceccant@infn.it
type      : RFC3820 compliant impersonation proxy
strength  : 1024
path      : /tmp/x509up_u501
timeleft  : 11:59:55
key usage : Digital Signature, Key Encipherment, Data Encipherment
=== VO wlcg extension information ===
VO        : wlcg
subject   : /DC=org/DC=terena/DC=tcs/C=IT/O=Istituto Nazionale di Fisica Nucleare/CN=Andrea Ceccanti aceccant@infn.it
issuer    : /DC=org/DC=terena/DC=tcs/C=IT/L=Frascati/O=Istituto Nazionale di Fisica Nucleare/CN=wlcg-voms.cloud.cnaf.infn.it
attribute : /wlcg
attribute : /wlcg/xfers
timeleft  : 11:59:55
uri       : wlcg-voms.cloud.cnaf.infn.it:15001
```

[wlcg-iam]: https://wlcg.cloud.cnaf.infn.it
[iam-docs]: https://indigo-iam.github.io/
[client-registration]:  https://indigo-iam.github.io/v/current/docs/tasks/user/client-registration/
[x509-linking]: https://indigo-iam.github.io/v/current/docs/tasks/user/x509-linking/ 
[oidc-agent]: https://github.com/indigo-dc/oidc-agent
