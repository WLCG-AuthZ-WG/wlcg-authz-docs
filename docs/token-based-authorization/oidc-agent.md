# OIDC Agent 

[oidc-agent](https://github.com/indigo-dc/oidc-agent) is a set of tools to get and manage OpenID Connect tokens and make them easily usable from the command line. It follows the ```ssh-agent``` design, so users can handle OIDC tokens in a similar way as they do with ssh keys.

## **Installation instructions**

Releases are available for several RPM and DEB based distribution, plus MacOS and Windows.
[The KIT Repo Server](https://repo.data.kit.edu) shows installation
instructions for each release

### **Example: Quick CENTOS7 installation recipe**

This example shows how to quickly install ```oidc-agent``` on CENTOS 7.



```
$ yum -y install epel-release

$ cd /etc/yum.repos.d; wget https://repo.data.kit.edu/data-kit-edu-centos7.repo

$ yum -y install oidc-agent
```


### **Bootstrapping oidc-agent**

The first thing to do is to start oidc-agent. This can be done issuing the following command:

```
$ eval $(oidc-agent)
Agent pid 62088
```

* ***To install OIDC Agent from source or in a Debian/Ubuntu distro, please refer to the [KIT Repo Server](https://repo.data.kit.edu) and to the oidc-agent [installation documentation in gitbook](https://indigo-dc.gitbook.io/oidc-agent/installation/install)***

## **How to register a client** 

In order to obtain a token out of an OP such as IAM, a user needs a client registered. oidc-agent can automate this step and store client credentials securely on the user machine.

A new client is registered using the ```oidc-gen``` command, as follows:

```
$ oidc-gen --iss https://wlcg.cloud.cnaf.infn.it --scope max --flow=device  wlcg
```
The ```--flow=device``` instructs ```oidc-agent``` to use the device code flow for the authentication, which is the recommended way with IAM.

oidc-agent will use "dynamic client registration" to register a new client and store the client credentials and a refresh token locally in encrypted form (the agent will ask for a password from the user).

Some IAM instance may not have "dynamic client registration" enabled. In this
case you may try `oidc-gen` with the `--pub` parameter.
This makes use of a pre-registered "public client", which may be available for that IAM instance.

## **How to print a list of all configured accounts** 

To obtain a list of all configured accounts configured, either ```oidc-gen --accounts``` or ```oidc-add --list``` can be used. Both of them can use the same flag ```-l```

```
$ oidc-gen -l
The following account configurations are usable: 
wlcg
```

## **How to print a client description** 

Printing the full client decrypted content can be done by passing the account shortname or the absolute filepath of the account, with ```oidc-gen --print``` or simply the ```-p``` flag


```
$ oidc-gen -p wlcg
Enter decryption password for account config 'wlcg': 
{
        "name": "wlcg",
        "client_name":  "oidc-agent:wlcg",
        "issuer_url":   "https://wlcg.cloud.cnaf.infn.it/",
        "device_authorization_endpoint":        "https://wlcg.cloud.cnaf.infn.it/devicecode",
        "daeSetByUser": 0,
        "client_id":    "f062c71e-920d-4b64-8282-a24d4101d8fc",
        "client_secret":        "xxxxxxxxxxxxxxxx",
        "refresh_token":        "xxxxxxxxxxxxxxxx",
        "cert_path":    "",
        "scope":        "address openid profile storage.read:/ wlcg eduperson_entitlement storage.create:/ phone offline_access eduperson_scoped_affiliation storage.modify:/ email wlcg.groups",
        "audience":     "",
        "redirect_uris":        ["edu.kit.data.oidc-agent:/redirect", "http://localhost:8080", "http://localhost:4242", "http://localhost:10088"],
        "username":     "",
        "password":     "",
        "client_id_issued_at":  1592322007,
        "registration_access_token":    "xxxxxxxxxxxxxx",
        "registration_client_uri":      "xxxxxxxxxxxxxx",
        "token_endpoint_auth_method":   "client_secret_basic",
        "grant_types":  ["urn:ietf:params:oauth:grant-type:device_code", "refresh_token"],
        "response_types":       ["token"],
        "application_type":     "web",
        "cert_path":    "",
        "audience":     ""
}
```

## **How to get a token from oidc-agent** 

Tokens can be obtained using the ```oidc-token``` command, as follows:

```
oidc-token wlcg
```

This will request a token with all the scopes requested at client registration time. 

### **Limiting issued scopes** 

To limit the scopes included in the token, the ```-s``` flag can be used, as follows:

```
oidc-token -s storage.read:/ wlcg
```

In this example the scopes is being limited to ```storage.read:/```

### **Limiting token audience** 

The token audience can be limited using the ```--aud``` flag,

```
oidc-token --aud example.audience -s storage.read:/ wlcg
```

In this example the audience is being defined as ```example.audience```

* ***For more usage options please refer to ```oidc-agent --help``` or to [oidc-agent usage documentation](https://indigo-dc.gitbook.io/oidc-agent/user)***
