# i4Trust development environment

Components for the development stage in namespace `i4trust-dev`.


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


### elasticsearch





## Marketplace

The marketplace consists of the Business API Ecosystem (BAE) and its own Keyrock 
instance.

### Keyrock

* DB setup incl. secret
Create a Secret manifest in the secrets/ folder:
```
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


* On UI: Create application, note down client-id/-secret

### BAE

* DB setup incl. secrets
* Add oauth2 credentials of BAE Keyrock instance as sealed-secret
* i4Trust related secrets for private key




## Packet Delivery Company




## Happy Pets




## No Cheaper



