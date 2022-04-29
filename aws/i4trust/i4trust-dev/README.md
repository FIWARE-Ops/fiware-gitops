# i4Trust development environment

Components for the development stage in namespace `i4trust-dev`.

## ServiceAccount

Some applications need root priviliges. They have to use the ServiceAccount created with the manifest 
[./serviceaccounts/i4trust-dev-root-runner.yaml](./serviceaccounts/i4trust-dev-root-runner.yaml).

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


### MongoDB

MongoDB is deployed with the bitnami chart.

* MongoDB root password
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

* MongoDB user passwords for init-db scripts
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


### elasticsearch

No further configuration required.



## Marketplace

The marketplace consists of the Business API Ecosystem (BAE) and its own Keyrock 
instance.

### Keyrock

* DB setup incl. secret
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


* Create Application for marketplace
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

* APIs
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

* RSS
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

* Charging Backend
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

* Logic Proxy
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


### Usage

The marketplace is accessible via the URL [https://i4trust-dev-bae.apps.fiware-dev-aws.fiware.dev/](https://i4trust-dev-bae.apps.fiware-dev-aws.fiware.dev/). 

The 'Local IDP' login option allows to login using the locally configured Keyrock IDP. In order to be able to perform administrative tasks on the 
marketplace, a user on the local Keyrock instance must be authorized in the marketplace application with the admin role.

As admin, one can also add further external IDPs, like for Packet Delivery or Happy Pets.




## Packet Delivery Company




## Happy Pets




## No Cheaper



