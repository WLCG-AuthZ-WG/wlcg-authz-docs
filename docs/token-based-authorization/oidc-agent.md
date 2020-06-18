# OIDC Agent 

[oidc-agent](https://github.com/indigo-dc/oidc-agent) is a set of tools to get and manage OpenID Connect tokens and make them easily usable from the command line. It follows the ```ssh-agent``` design, so users can handle OIDC tokens in a similar way as they do with ssh keys.

## **Installation instructions**

### **Quick CENTOS7 installation recipe**

Current releases are available at [GitHub](https://github.com/indigo-dc/oidc-agent/releases)

This recipe shows how to quickly install ```oidc-agent``` on CENTOS 7.

```
$ VERSION=$(curl --silent "https://api.github.com/repos/indigo-dc/oidc-agent/releases/latest" | grep -Po '"tag_name": "v\K.*?(?=")') '"tag_name": "v\K.*?(?=")'

$ yum -y install epel-release

$ yum -y install https://github.com/indigo-dc/oidc-agent/releases/download/v$VERSION/oidc-agent-$VERSION-1.el7.x86_64.rpm
```

### **Bootstrapping oidc-agent**

The first thing to do is to start oidc-agent. This can be done issuing the following command:

```
$ eval $(oidc-agent)
Agent pid 62088
```

* ***To install OIDC Agent from source or in a Debian/Ubuntu distro, please refer to oidc-agent [instalation documentation](https://indigo-dc.gitbook.io/oidc-agent/installation/install)***

## **How to register a client** 

In order to obtain a token out of IAM, a user needs a client registered. oidc-agent can automate this step and store client credentials securely on the user machine.

A new client can be registered using the ```oidc-gen``` command, as follows:

```
$ oidc-gen -w device wlcg
```
The ```-w device``` instructs ```oidc-agent``` to use the device code flow for the authentication, which is the recommended way with IAM.

oidc-agent will display a list of different providers that can be used for registration:

```
[1] https://iam-test.indigo-datacloud.eu/
[2] https://iam.deep-hybrid-datacloud.eu/
...
[16] https://wlcg.cloud.cnaf.infn.it/
Issuer [https://wlcg.cloud.cnaf.infn.it/]:_
```

Select one of the registered providers, or type a custom issuer (for IAM, the last character of the issuer string is always a /, e.g. https://wlcg.cloud.cnaf.infn.it/).

Then oidc-agent asks for the scopes, typing max (without quotes) allows to get all the allowed scopes, or, one or more can be defined using spaces between them.

oidc-agent will register a new client and store the client credentials and a refresh token locally in encrypted form (the agent will ask for a password from the user).

## **How to print a list of all configured account** 

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