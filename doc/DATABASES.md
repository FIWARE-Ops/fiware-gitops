# Databases

In order to allow seperate databases from the actual cluster, we are using managed services provided by AWS. Openshift has a couple of Community Operators for that.
The following chapters describe the steps executed to have automatic support for managed databases in this cluster.

## General pre-requesites

In general, follow the steps described in the [aws-documentation](https://aws-controllers-k8s.github.io/community/docs/user-docs/openshift/):

1. Setup the [```ack-system```-namespace](../aws/namespaces/ack-system.yaml)

2. Create a user:
```shell
    aws iam create-user --user-name ack-service-controller
```

3. Enable programmatic access:
```shell
    aws iam create-access-key --user-name ack-service-controller
```

4. Create the user-config(see [aws-doc for more](https://aws-controllers-k8s.github.io/community/docs/user-docs/openshift/)) - [ack-systems/user-config](../aws/ack-system/user-config/)

5. Create the user-secrets, using the access-key created in the previous step(see [aws-doc for more](https://aws-controllers-k8s.github.io/community/docs/user-docs/openshift/)) - [ack-systems/user-config](../aws/ack-system/secrets/) 

## Install RDS Controller

The [RDS-Controller](https://github.com/aws-controllers-k8s/rds-controller) is responsible for providing different (SQL)-Databases to the cluster.

1. Add policy to AWS user:
```shell
    aws iam attach-user-policy \
        --user-name ack-service-controller \
        --policy-arn 'arn:aws:iam::aws:policy/AmazonRDSFullAccess'
```

2. Create configmap for service

1. Select the controller:

![Search](./aws-services/rds-controller-search.png)

2. Install with defaults:

![Install](./aws-services/rds-controller-install.png)

3. Deploy an rds-resource - Example:
```yaml
apiVersion: rds.services.k8s.aws/v1alpha1
kind: DBInstance
metadata:
  name: mysql
spec:
  allocatedStorage: 20
  dbInstanceClass: db.t3.micro
  dbInstanceIdentifier: "mysql"
  engine: mysql
  engineVersion: 5.7.31
  masterUsername: admin
  masterUserPassword:
    namespace: default
    name: password-secret
    key: password
```