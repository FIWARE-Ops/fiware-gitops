# i4Trust development environment

Components for the development stage in namespace `i4trust-dev`.


## Databases


### MySQL

MySQL is used with the t3n chart.

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



## Marketplace

The marketplace consists of the Business API Ecosystem (BAE) and its own Keyrock 
instance.

### Keyrock

* DB setup incl. secret
* On UI: Create application, note down client-id/-secret

### BAE

* DB setup incl. secrets
* Add oauth2 credentials of BAE Keyrock instance as sealed-secret
* i4Trust related secrets for private key




## Packet Delivery Company




## Happy Pets




## No Cheaper



