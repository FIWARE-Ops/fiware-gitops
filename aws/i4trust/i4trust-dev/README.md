# i4Trust development environment

Components for the development stage in namespace `i4trust-dev`.

For deployment and configuration of all components, follow the order below.

The corresponding ArgoCD apps can be 
found [here](../../apps/i4trust/i4trust-dev). 
Apps can be created with the argocd CLI:
```shell
argocd app create -f <APPLICATION_YAML>
```




## ServiceAccount

Some applications need root priviliges. They have to use the ServiceAccount created with the manifest 
[./serviceaccounts/i4trust-dev-root-runner.yaml](./serviceaccounts/i4trust-dev-root-runner.yaml).
For this, first create the app 
[i4trust-dev-serviceaccounts](../../apps/i4trust/i4trust-dev/i4trust-dev-serviceaccounts.yaml).

When the ServiceAccount was created within the namespace, add it to the privileged users:
```shell
oc edit scc privileged
```
In the open editor, add the new service account under `users:`, e.g.
```yaml
users:
- system:admin
- ...
- system:serviceaccount:i4trust-dev:i4trust-dev-root-runner
- ...
```

In case of an error when patching, there was a `yaml` file created under `/tmp`. Enforce the replacement by e.g.
```shell
oc replace -f /tmp/oc-edit-pzvsn.yaml
```


## Secrets

In the following, several secrets are created using the 
[sealed-secrets project](https://github.com/FIWARE-Ops/fiware-gitops#7-deploy-bitnamisealed-secrets). 
In general, [created secrets](https://github.com/FIWARE-Ops/fiware-gitops#8-create-secrets) get encrypted before 
pushing them to GitHub.

For deployment of all sealed secrets, create the ArgoCD 
app [i4trust-dev-secrets](../../apps/i4trust/i4trust-dev/i4trust-dev-secrets.yaml).

Note, that the sealed-secrets within this repository can only be decrypted by the sealed-secrets 
controller deployed at the FIWARE OpenShift cluster. When deploying on your own infrastructure, 
you need to create your own sealed-secrets first, as described below for the different 
components.



## ConfigMaps

ConfigMaps are used to store additional configuration data used by the components, or initial data 
for databases and the Context Broker.

For deployment of all ConfigMaps, create the ArgoCD 
app [i4trust-dev-configmaps](../../apps/i4trust/i4trust-dev/i4trust-dev-configmaps.yaml).



## Jobs

Jobs are used to perform initialisation tasks, like creating initial entities at the Context Broker.

For deployment of all jobs, create the ArgoCD 
app [i4trust-dev-jobs](../../apps/i4trust/i4trust-dev/i4trust-dev-jobs.yaml).




## Databases


### MySQL

MySQL is deployed with the t3n chart.

To create an own secret, change to the secrets/ folder and create a 
secret manifest `mysql-secret.manifest.yaml`:
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: mysql-secret
  namespace: i4trust-dev
data:
  # Your base64 encoded root password
  mysql-root-password: <BASE64_PASSWORD>
```

Create a sealed secret with `kubeseal`:
```shell
kubeseal <mysql-secret.manifest.yaml >mysql-sealed-secret.yaml -o yaml --controller-namespace sealed-secrets --controller-name sealed-secrets
```

Then create the corresponding ArgoCD app.



### MongoDB

MongoDB is deployed with the bitnami chart.

#### MongoDB root password
To create an own secret, change to the secrets/ folder and create a 
secret manifest `mongodb-secret.manifest.yaml`:
```yaml
apiVersion: v1
kind: Secret
metadata:
  # name of the secret
  name: mongodb-secret
  # namespace the secret should be deployed to - important, sealed-secrets will check the namespace before decryption
  namespace: i4trust-dev
data:
  # the actual data, needs to be base64 encoded
  mongodb-password: <BASE64_PASSWORD>
  mongodb-replica-set-key: <BASE64_PASSWORD>
  mongodb-root-password: <BASE64_PASSWORD>
```

Create a sealed secret with `kubeseal`:
```shell
kubeseal <mongodb-secret.manifest.yaml >mongodb-sealed-secret.yaml -o yaml --controller-namespace sealed-secrets --controller-name sealed-secrets
```

#### MongoDB user passwords for init-db scripts
Create a secret manifest `mongodb-envs-secret.manifest.yaml`:
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: mongodb-envs-secret
  namespace: i4trust-dev
data:
  # Base64 encoded PW for charging user
  MONGO_BAE_CHARGING_PW: <BASE64_PASSWORD>
  # Base64 encoded PW for belp user
  MONGO_BAE_BELP_PW: <BASE64_PASSWORD>
```

Create a sealed secret:
```shell
kubeseal <mongodb-envs-secret.manifest.yaml >mongodb-envs-sealed-secret.yaml -o yaml --controller-namespace sealed-secrets --controller-name sealed-secrets
```

#### App

Then create the corresponding ArgoCD app for MongoDB.



### elasticsearch

No further configuration required. Just create the corresponding ArgoCD app.



## Satellite

The trust anchor satellite is based on the iSHARE Satellite, and is required to ensure trust among all 
participants of the data space.


### Secret

Create a Secret manifest `satellite-secret.manifest.yaml` in the secrets/ folder:
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: satellite-secret
  namespace: i4trust-dev
data:
  # Base64 encoded cert chain PEM
  SATELLITE_CERTS: <BASE64_CERT> 
  # Base64 encoded private key PEM
  SATELLITE_KEY: <BASE64_KEY>
```
where the cert chain and key were created and issued for the satellite

Create a sealed secret with `kubeseal`:
```shell
kubeseal <satellite-secret.manifest.yaml >satellite-sealed-secret.yaml -o yaml --controller-namespace sealed-secrets --controller-name sealed-secrets
```

### ArgoCD app

Create the corresponding ArgoCD app



## Marketplace

The marketplace consists of the Business API Ecosystem (BAE) and its own Keyrock 
instance. It will use the previously deployed databases.


### Keyrock

#### Secret

Create a Secret manifest `bae-keyrock-secret.manifest.yaml` in the secrets/ folder:
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: bae-keyrock-secret
  namespace: i4trust-dev
data:
  # Base64 encoded
  adminPassword: <BASE64_ADMIN_PASSWORD>
  dbPassword: <BASE64_DB_PASSWORD>
```
where `dbPassword` must match to the root password of the MySQL.

Create a sealed secret with `kubeseal`:
```shell
kubeseal <bae-keyrock-secret.manifest.yaml >bae-keyrock-sealed-secret.yaml -o yaml --controller-namespace sealed-secrets --controller-name sealed-secrets
```

#### App

Create the corresponding ArgoCD app.


#### Create Application for marketplace in Keyrock

Login on [Keyrock](https://i4trust-dev-bae-keyrock.apps.fiware-dev-aws.fiware.dev/) with the admin credentials and create a new application.

Fill in the following values:
```yaml
Name: i4Trust Marketplace
Description: "Enter some description"
URL: https://i4trust-dev-bae.apps.fiware-dev-aws.fiware.dev
Callback-URL: http://i4trust-dev-bae.apps.fiware-dev-aws.fiware.dev/auth/fiware/callback
Grant-Type: "Authorization Code, Refresh Token"
```
and add the follwing roles:
```yaml
roles:
  - admin
  - orgAdmin
  - seller
  - customer
```

When the application was created, note down the client ID and client secret.




### BAE

The BAE consists of sifferent components running as separate services:
* TMForum APIs
* RSS
* Charging Backend
* Logic Proxy

Each requires further configuration.


#### APIs

Create a Secret manifest `bae-apis-secret.manifest.yaml` in the secrets/ folder:
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: bae-apis-secret
  namespace: i4trust-dev
data:
  # Base64 encoded
  dbPassword: <BASE64_DB_PASSWORD>
```
where `dbPassword` must match to the root password of the MySQL.

Create a sealed secret with `kubeseal`:
```shell
kubeseal <bae-apis-secret.manifest.yaml >bae-apis-sealed-secret.yaml -o yaml --controller-namespace sealed-secrets --controller-name sealed-secrets
```

#### RSS

Create a Secret manifest `bae-rss-secret.manifest.yaml` in the secrets/ folder:
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: bae-rss-secret
  namespace: i4trust-dev
data:
  # Base64 encoded
  dbPassword: <BASE64_DB_PASSWORD>
```
where `dbPassword` must match to the root password of the MySQL.

Create a sealed secret with `kubeseal`:
```shell
kubeseal <bae-rss-secret.manifest.yaml >bae-rss-sealed-secret.yaml -o yaml --controller-namespace sealed-secrets --controller-name sealed-secrets
```

#### Charging Backend

Create a Secret manifest `bae-cb-secret.manifest.yaml` in the secrets/ folder:
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: bae-cb-secret
  namespace: i4trust-dev
data:
  # MongoDB charing user password
  dbPassword: <BASE64_PASSWORD>
  # PayPal client secret
  paypalClientSecret: <BASE64_CLIENT_SECRET>
  # Password of SMTP (dummy)
  smtpPassword: <BASE64_DUMMY>
  # IDP password for plugins (dummy)
  pluginsIdmPassword: <BASE64_DUMMY>
```
where `dbPassword` must match to the charging user password of the MongoDB. Dummys can be set with any base64 encoded string.

Create a sealed secret with `kubeseal`:
```shell
kubeseal <bae-cb-secret.manifest.yaml >bae-cb-sealed-secret.yaml -o yaml --controller-namespace sealed-secrets --controller-name sealed-secrets
```

Create a Secret manifest `bae-cb-cert-secret.manifest.yaml` in the secrets/ folder:
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: bae-cb-cert-secret
  namespace: i4trust-dev
data:
  # Base64 encoded cert chain PEM
  cert.pem: <BASE64_CERT> 
  key.pem: <BASE64_KEY>
```
where the cert chain and key were issued to the marketplace.

Create a sealed secret with `kubeseal`:
```shell
kubeseal <bae-cb-cert-secret.manifest.yaml >bae-cb-cert-sealed-secret.yaml -o yaml --controller-namespace sealed-secrets --controller-name sealed-secrets
```

#### Logic Proxy
Make sure, that in the BAE [values.yaml](./bae/values.yaml) the client ID is entered which was generated in the local Keyrock IDP application before.

Create a Secret manifest `bae-lp-secret.manifest.yaml` in the secrets/ folder:
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: bae-lp-secret
  namespace: i4trust-dev
data:
  # MongoDB charing user password
  dbPassword: <BASE64_PASSWORD>
  # OAuth2 client secret of local Keyrock IDP Application
  oauthClientSecret: <BASE64_CLIENT_SECRET>
```
where `oauthClientSecret` is the client secret which was generated in the local Keyrock IDP application before.

Create a sealed secret with `kubeseal`:
```shell
kubeseal <bae-lp-secret.manifest.yaml >bae-lp-sealed-secret.yaml -o yaml --controller-namespace sealed-secrets --controller-name sealed-secrets
```

Create a Secret manifest `bae-lp-cert-secret.manifest.yaml` in the secrets/ folder:
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: bae-lp-cert-secret
  namespace: i4trust-dev
data:
  # Base64 encoded cert chain PEM
  cert.pem: <BASE64_CERT> 
  key.pem: <BASE64_KEY>
```
where the cert chain and key were issued to the marketplace.

Create a sealed secret with `kubeseal`:
```shell
kubeseal <bae-lp-cert-secret.manifest.yaml >bae-lp-cert-sealed-secret.yaml -o yaml --controller-namespace sealed-secrets --controller-name sealed-secrets
```


#### ArgoCD App

Create the ArgoCD app [i4trust-dev-bae](../../apps/i4trust/i4trust-dev/i4trust-dev-bae.yaml).



#### Plugins

Plugins need to be installed on the charging backend after its deployment.

In order to install a plugin, perform the following steps:

Create zip file from plugin directory
```shell
zip -r my-plugin.zip my-plugin/
```

Copy to plugin directory of charging backend pod
```shell
kubectl cp ./my-plugin.zip my-charging-backend-pod:/opt/business-ecosystem-charging-backend/src/plugins/.
```

Open shell in Pod and load plugin
```shell
kubectl exec --stdin --tty my-charging-backend-pod -- /bin/bash

# Shell within pod:
root@my-charging-backend-pod:/business-ecosystem-charging-backend/src# ./manage.py loadplugin plugins/my-plugin.zip
```

In order to enable the creation of assets for data services in i4Trust, 
the [bae-i4trust-service plugin](https://github.com/i4Trust/bae-i4trust-service) needs to be installed.



### Usage

The marketplace is accessible via the URL [https://i4trust-dev-bae.apps.fiware-dev-aws.fiware.dev/](https://i4trust-dev-bae.apps.fiware-dev-aws.fiware.dev/). 

The 'Local IDP' login option allows to login using the locally configured Keyrock IDP. In order to be able to perform administrative tasks on the 
marketplace, a user on the local Keyrock instance must be authorized in the marketplace application with the admin role.

As admin, one can also add further external IDPs, like for Packet Delivery or Happy Pets.




## Packet Delivery Company

The environment of Packet Delivery Company consists of:
* Keyrock IDP
* Orion-LD Context Broker
* Activation Service (in addition to Authorization Registry)
* PEP/PDP Gateway (Kong)

An external Authorisation Registry is configured. Components are using 
the previously deployed databases.



### Keyrock

#### Secrets

Create a Secret manifest `pdc-keyrock-secret.manifest.yaml` in the secrets/ folder:
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: pdc-keyrock-secret
  namespace: i4trust-dev
data:
  # Base64 encoded
  adminPassword: <BASE64_ADMIN_PASSWORD>
  dbPassword: <BASE64_DB_PASSWORD>
```
where `dbPassword` must match to the root password of the MySQL.

Create a sealed secret with `kubeseal`:
```shell
kubeseal <pdc-keyrock-secret.manifest.yaml >pdc-keyrock-sealed-secret.yaml -o yaml --controller-namespace sealed-secrets --controller-name sealed-secrets
```

Create a Secret manifest `pdc-keyrock-cert-secret.manifest.yaml` in the secrets/ folder:
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: pdc-keyrock-cert-secret
  namespace: i4trust-dev
data:
  # Base64 encoded cert chain PEM
  cert: <BASE64_CERT> 
  # Base64 encoded private key PEM
  key: <BASE64_KEY>
```
where the cert chain and key were issued to the Packet Delivery Company.

Create a sealed secret with `kubeseal`:
```shell
kubeseal <pdc-keyrock-cert-secret.manifest.yaml >pdc-keyrock-cert-sealed-secret.yaml -o yaml --controller-namespace sealed-secrets --controller-name sealed-secrets
```


#### ArgoCD app

Create the ArgoCD app [i4trust-dev-pdc-keyrock](../../apps/i4trust/i4trust-dev/i4trust-dev-pdc-keyrock.yaml).



### Orion

#### Secret

Create a Secret manifest `pdc-orion-secret.manifest.yaml` in the secrets/ folder:
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: pdc-orion-secret
  namespace: i4trust-dev
data:
  # MongoDB root user password
  dbPassword: <BASE64_PASSWORD>
```
where `dbPassword` must match to the root password of the MongoDB.

Create a sealed secret with `kubeseal`:
```shell
kubeseal <pdc-orion-secret.manifest.yaml >pdc-orion-sealed-secret.yaml -o yaml --controller-namespace sealed-secrets --controller-name sealed-secrets
```

#### ArgoCD app

Create the corresponding ArgoCD app



### Kong

#### Secret

Create a Secret manifest `pdc-kong-secret.manifest.yaml` in the secrets/ folder:
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: pdc-kong-secret
  namespace: i4trust-dev
data:
  # Base64 encoded cert chain PEM
  FIWARE_JWS_X5C: <BASE64_CERT> 
  # Base64 encoded private key PEM
  FIWARE_JWS_PRIVATE_KEY: <BASE64_KEY>
```
where the cert chain and key were issued to the Packet Delivery Company.

Create a sealed secret with `kubeseal`:
```shell
kubeseal <pdc-kong-secret.manifest.yaml >pdc-kong-sealed-secret.yaml -o yaml --controller-namespace sealed-secrets --controller-name sealed-secrets
```

#### ArgoCD app

Create the corresponding ArgoCD app



### Activation Service

The activation service is required, so that the marketplace is able to create policies at the AR of Packet Delivery.

#### Secret

Create a Secret manifest `pdc-as-secret.manifest.yaml` in the secrets/ folder:
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: pdc-as-secret
  namespace: i4trust-dev
data:
  # Base64 encoded cert chain PEM
  AS_CLIENT_CRT: <BASE64_CERT> 
  # Base64 encoded private key PEM
  AS_CLIENT_KEY: <BASE64_KEY>
```
where the cert chain and key were issued to the Packet Delivery Company.

Create a sealed secret with `kubeseal`:
```shell
kubeseal <pdc-as-secret.manifest.yaml >pdc-as-sealed-secret.yaml -o yaml --controller-namespace sealed-secrets --controller-name sealed-secrets
```

#### ArgoCD app

Create the corresponding ArgoCD app




### Packet Delivery Service Portal

The portal is a simple demo application, which showcases, how a login is performed via an external IDP in i4Trust, 
and how requests are sent to a data service with an access token JWT.

#### Secret

Create a Secret manifest `pdc-portal-secret.manifest.yaml` in the secrets/ folder:
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: pdc-portal-secret
  namespace: i4trust-dev
data:
  # Base64 encoded cert chain PEM
  PORTAL_CLIENT_CRT: <BASE64_CERT> 
  # Base64 encoded private key PEM
  PORTAL_CLIENT_KEY: <BASE64_KEY>
```
where the cert chain and key were issued to the Packet Delivery Company.

Create a sealed secret with `kubeseal`:
```shell
kubeseal <pdc-portal-secret.manifest.yaml >pdc-portal-sealed-secret.yaml -o yaml --controller-namespace sealed-secrets --controller-name sealed-secrets
```


#### ArgoCD app

Create the corresponding ArgoCD app







## Happy Pets

The environment of Happy Pets consists of two instance of the Keyrock IDP. One is dedicated to company employees, the other instances is used 
for the users of the shop.


### Keyrock (Company)

#### Secrets

Create a Secret manifest `happypets-keyrock-secret.manifest.yaml` in the secrets/ folder:
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: happypets-keyrock-secret
  namespace: i4trust-dev
data:
  # Base64 encoded
  adminPassword: <BASE64_ADMIN_PASSWORD>
  dbPassword: <BASE64_DB_PASSWORD>
```
where `dbPassword` must match to the root password of the MySQL.

Create a sealed secret with `kubeseal`:
```shell
kubeseal <happypets-keyrock-secret.manifest.yaml >happypets-keyrock-sealed-secret.yaml -o yaml --controller-namespace sealed-secrets --controller-name sealed-secrets
```

Create a Secret manifest `happypets-keyrock-cert-secret.manifest.yaml` in the secrets/ folder:
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: happypets-keyrock-cert-secret
  namespace: i4trust-dev
data:
  # Base64 encoded cert chain PEM
  cert: <BASE64_CERT> 
  # Base64 encoded private key PEM
  key: <BASE64_KEY>
```
where the cert chain and key were issued to the Packet Delivery Company.

Create a sealed secret with `kubeseal`:
```shell
kubeseal <happypets-keyrock-cert-secret.manifest.yaml >happypets-keyrock-cert-sealed-secret.yaml -o yaml --controller-namespace sealed-secrets --controller-name sealed-secrets
```


#### ArgoCD app

Create the corresponding ArgoCD app



### Keyrock (Shop)

#### Secrets

Create a Secret manifest `happypets-keyrock-shop-secret.manifest.yaml` in the secrets/ folder:
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: happypets-keyrock-shop-secret
  namespace: i4trust-dev
data:
  # Base64 encoded
  adminPassword: <BASE64_ADMIN_PASSWORD>
  dbPassword: <BASE64_DB_PASSWORD>
```
where `dbPassword` must match to the root password of the MySQL.

Create a sealed secret with `kubeseal`:
```shell
kubeseal <happypets-keyrock-shop-secret.manifest.yaml >happypets-keyrock-shop-sealed-secret.yaml -o yaml --controller-namespace sealed-secrets --controller-name sealed-secrets
```

Create a Secret manifest `happypets-keyrock-shop-cert-secret.manifest.yaml` in the secrets/ folder:
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: happypets-keyrock-shop-cert-secret
  namespace: i4trust-dev
data:
  # Base64 encoded cert chain PEM
  cert: <BASE64_CERT> 
  # Base64 encoded private key PEM
  key: <BASE64_KEY>
```
where the cert chain and key were issued to the Packet Delivery Company.

Create a sealed secret with `kubeseal`:
```shell
kubeseal <happypets-keyrock-shop-cert-secret.manifest.yaml >happypets-keyrock-shop-cert-sealed-secret.yaml -o yaml --controller-namespace sealed-secrets --controller-name sealed-secrets
```


#### ArgoCD app

Create the corresponding ArgoCD app



#### Add as external IDP to BAE

The Keyrock company instance should be added as external IDP to the marketplace.

Login as admin on the BAE and open the administrative UI.

Add an external IDP with the EORI of Happy Pets and the hostname of the company Keyrock 
IDP ([https://i4trust-dev-happypets-keyrock.apps.fiware-dev-aws.fiware.dev](https://i4trust-dev-happypets-keyrock.apps.fiware-dev-aws.fiware.dev)).






## No Cheaper

The environment of No CHeaper consists of two instance of the Keyrock IDP. One is dedicated to company employees, the other instances is used 
for the users of the shop.


### Keyrock (Company)

#### Secret

Create a Secret manifest `nocheaper-keyrock-secret.manifest.yaml` in the secrets/ folder:
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: nocheaper-keyrock-secret
  namespace: i4trust-dev
data:
  # Base64 encoded
  adminPassword: <BASE64_ADMIN_PASSWORD>
  dbPassword: <BASE64_DB_PASSWORD>
```
where `dbPassword` must match to the root password of the MySQL.

Create a sealed secret with `kubeseal`:
```shell
kubeseal <nocheaper-keyrock-secret.manifest.yaml >nocheaper-keyrock-sealed-secret.yaml -o yaml --controller-namespace sealed-secrets --controller-name sealed-secrets
```

Create a Secret manifest `nocheaper-keyrock-cert-secret.manifest.yaml` in the secrets/ folder:
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: nocheaper-keyrock-cert-secret
  namespace: i4trust-dev
data:
  # Base64 encoded cert chain PEM
  cert: <BASE64_CERT> 
  # Base64 encoded private key PEM
  key: <BASE64_KEY>
```
where the cert chain and key were issued to the Packet Delivery Company.

Create a sealed secret with `kubeseal`:
```shell
kubeseal <nocheaper-keyrock-cert-secret.manifest.yaml >nocheaper-keyrock-cert-sealed-secret.yaml -o yaml --controller-namespace sealed-secrets --controller-name sealed-secrets
```


#### ArgoCD app

Create the corresponding ArgoCD app





### Keyrock (Shop)

#### Secret

Create a Secret manifest `nocheaper-keyrock-shop-secret.manifest.yaml` in the secrets/ folder:
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: nocheaper-keyrock-shop-secret
  namespace: i4trust-dev
data:
  # Base64 encoded
  adminPassword: <BASE64_ADMIN_PASSWORD>
  dbPassword: <BASE64_DB_PASSWORD>
```
where `dbPassword` must match to the root password of the MySQL.

Create a sealed secret with `kubeseal`:
```shell
kubeseal <nocheaper-keyrock-shop-secret.manifest.yaml >nocheaper-keyrock-shop-sealed-secret.yaml -o yaml --controller-namespace sealed-secrets --controller-name sealed-secrets
```

Create a Secret manifest `nocheaper-keyrock-shop-cert-secret.manifest.yaml` in the secrets/ folder:
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: nocheaper-keyrock-shop-cert-secret
  namespace: i4trust-dev
data:
  # Base64 encoded cert chain PEM
  cert: <BASE64_CERT> 
  # Base64 encoded private key PEM
  key: <BASE64_KEY>
```
where the cert chain and key were issued to the Packet Delivery Company.

Create a sealed secret with `kubeseal`:
```shell
kubeseal <nocheaper-keyrock-shop-cert-secret.manifest.yaml >nocheaper-keyrock-shop-cert-sealed-secret.yaml -o yaml --controller-namespace sealed-secrets --controller-name sealed-secrets
```


#### ArgoCD app

Create the corresponding ArgoCD app



#### Add as external IDP to BAE

The Keyrock company instance should be added as external IDP to the marketplace.

Login as admin on the BAE and open the administrative UI.

Add an external IDP with the EORI of No CHeaper and the hostname of the company Keyrock 
IDP ([https://i4trust-dev-nocheaper-keyrock.apps.fiware-dev-aws.fiware.dev](https://i4trust-dev-nocheaper-keyrock.apps.fiware-dev-aws.fiware.dev)).
