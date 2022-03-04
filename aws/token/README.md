# Test deployment for Token-Project

In order to support the development and testing in the [Token-Project](https://token-project.eu/the-project/) this namespace provides components for persisting transactions on [NGSI-LD entities](https://docbox.etsi.org/isg/cim/open/Latest%20release%20NGSI-LD%20API%20for%20public%20comment.pdf)
in a block-chain([alastria](https://alastria.io/en/)). 

> :warning: While this installation aims to be close to production environments, it makes a couple of compromises that should be avoided in real enviornments:
> - the API's are unsecured
> - it uses the [defaultAccount-Feature of canis-major](https://github.com/FIWARE/CanisMajor/blob/master/src/main/java/org/fiware/canismajor/configuration/DefaultAccountProperties.java#L10-L14), see [Signature-delegation ADR](https://github.com/FIWARE/CanisMajor/blob/master/docs/adrs/delegate-signatur.md) for the alternative
> - the installation is very small, thus might not be sufficient for production load

## Components

- [Orion-LD ContextBroker](https://github.com/FIWARE/context.Orion-LD) for persisting transaction receipts in [NGSI-LD](https://docbox.etsi.org/isg/cim/open/Latest%20release%20NGSI-LD%20API%20for%20public%20comment.pdf)
- [MongoDB](https://www.mongodb.com) as a storage backend for Orion-LD
- [Canis-Major](https://github.com/FIWARE/CanisMajor) as the central api for handling the transactions

## Example usage

1. Create an entity: 
```shell
curl --location --request POST 'https://token-canis-major-token.apps.fiware-dev-aws.fiware.dev/ngsi-ld/v1/entities' \
--header 'Content-Type: application/ld+json' \
--header 'Accept: application/json' \
--data-raw '{
    "id": "urn:ngsi-ld:Delivery:packet-1",
    "type": "Delivery",
    "name": {
    	"type": "Property",
        "value": "P-1"
    },
    "deliveryMethod": {
    	"type": "Property",
        "value": "airDrop"
    },
    "status": {
    	"type": "Property",
        "value": "created"
    },
    "@context": [
        "https://fiware.github.io/data-models/context.jsonld"
    ]
}'
```

2. Update the entity:
```shell
curl --location --request POST 'https://token-canis-major-token.apps.fiware-dev-aws.fiware.dev/ngsi-ld/v1/entities/urn:ngsi-ld:Delivery:packet-1/attrs' \
--header 'Content-Type: application/ld+json' \
--header 'Accept: application/json' \
--data-raw '{
    "requestedTimeslot": {
    	"type": "Property",
        "value": "2021-12-25T12:30:00+00:00"
    },
    "status": {
    	"type": "Property",
        "value": "awaiting_timeslot_confirmation"
    },
    "@context": [
        "https://fiware.github.io/data-models/context.jsonld"
    ]
}'
```
3. Upsert the entity and create an additional one:
```shell
curl --location --request POST 'https://token-canis-major-token.apps.fiware-dev-aws.fiware.dev/ngsi-ld/v1/entityOperations/upsert' \
--header 'Content-Type: application/ld+json' \
--header 'Accept: application/json' \
--header 'Cookie: 737e0c5d7968cbf5dc4309b4999b9618=5b1d409221b3d78f3fca36e26780146e' \
--data-raw '[{
    "id": "urn:ngsi-ld:Delivery:packet-1",
    "type": "Delivery",
    "name": {
    	"type": "Property",
        "value": "P-1"
    },
    "requestedTimeslot": {
    	"type": "Property",
        "value": "2021-12-25T12:30:00+00:00"
    },
    "deliveryMethod": {
    	"type": "Property",
        "value": "airDrop"
    },
    "status": {
    	"type": "Property",
        "value": "awaiting_delivery_confirmation"
    },
    "@context": [
        "https://fiware.github.io/data-models/context.jsonld"
    ]
},
{
    "id": "urn:ngsi-ld:Delivery:packet-2",
    "type": "Delivery",
    "name": {
    	"type": "Property",
        "value": "P-2"
    },
    "status": {
    	"type": "Property",
        "value": "created"
    },
    "@context": [
        "https://fiware.github.io/data-models/context.jsonld"
    ]
}]'
```
4. Get the transaction history:
   * For all entities: ```curl -X GET https://token-canis-major-token.apps.fiware-dev-aws.fiware.dev/entitiy```
   * For packet-1: ```curl -X GET https://token-canis-major-token.apps.fiware-dev-aws.fiware.dev/entitiy/urn:ngsi-ld:Delivery:packet-1```
   * For packet-2: ```curl -X GET https://token-canis-major-token.apps.fiware-dev-aws.fiware.dev/entitiy/urn:ngsi-ld:Delivery:packet-2```
 
