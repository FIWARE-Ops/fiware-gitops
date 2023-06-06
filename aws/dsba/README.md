# Demo-Setup DSBA-compliant Dataspace

This folder and its corresponding namespace contain a [DSBA-compliant](https://data-spaces-business-alliance.eu/wp-content/uploads/dlm_uploads/Data-Spaces-Business-Alliance-Technical-Convergence-V2.pdf) DataSpace for demonstrational purposes. The DataSpace is build with [FIWARE-Components](https://github.com/FIWARE), using parts of the [i4Trust-Framework](https://github.com/i4Trust) and the [Gaia-X Compliance Services](https://gitlab.com/gaia-x/lab/compliance).

## The DataSpace

![participants](docs/participants.png)

The DataSpace contains 5 particpants in various roles:

- the Marketplace, connecting consumers and providers of Data-Services
- Packet Delivery Co. as a Provider, offering Dataservices around their "traditional" delivery services
- HappyPets Inc. and Animal Goods Org. as Consumers, selling pet-related goods to end-customers using the serivces of Packet Delivery Co.
- the DataSpace Operator as a Trust Anchor, providing validation of participants and functionality to onboard to the dataspace

As an additional actor, outside of the Dataspace, the [Gaia-X Compliance Services](https://gitlab.com/gaia-x/lab/compliance) are used by the Dataspace Operator to validate Self-Descriptions of potential new participants during the OnBoarding.


## The Dataspace Operator

The Dataspace Operator acts as the TrustAnchor of the Dataspace. It provides the particpants information on whom to trust. 

To allow self-registration of new participants, the [OnBoarding-Portal](https://onboarding-portal.dsba.fiware.dev) is provided. "OnBoarding" means, that a participant is listed as a trusted participant at the Dataspace's [Trusted Issuers Registry](https://api-pilot.ebsi.eu/docs/apis/trusted-issuers-registry/latest#/). In case of the OnBoarding-Portal, the registry is backed by an [NGSI-LD](https://www.etsi.org/deliver/etsi_gs/CIM/001_099/009/01.06.01_60/gs_cim009v010601p.pdf) ContextBroker, which stores the required information about the participants. The ContextBroker API to store information about the participants is secured with a full DSBA-compliant security framework, therefor a number of components are deployed for the Dataspace Operator:

![operator](docs/onboarding.png)


The components are:

- the Portal as a central GUI, where Legal Representatives of potential participants can log-in and register for the Dataspace
- the Verifier to provide the authentication capabilities to the portal, so that the Legal Representative can use its mobile wallet to authenticate itself
- the Credentials Config Service to provide the Verifier with information about the scope to be requested from the wallet and the trust-anchors(e.g. Trusted Issuers List and Trusted Particpants List) to be used for the credential types
- the Trusted Issuers List that will be used by the Verifier to check that the Issuer is allowed to issue certain credentials and claims
- Walt-Id to verify variuous parts of the credential(f.e. the signature)
- Kong to enforce authorization on the requests towards Orion-LD
- the PDP to decide on the requests towards Orion-LD
- the Authorization Registry to provide the policies, used by the PDP
- Orion-LD to actually store information about the participants
- the Trusted Participants Registry to provide information about the Trusted Participants through an [EBSI-complinat Trusted-Issuers-Registry API](https://api-pilot.ebsi.eu/docs/apis/trusted-issuers-registry/latest#/)

## The Particpants

In the Dataspace, there are two types of participants "Consumers" and "Providers". While a particpant certainly can act in both roles, the demo-environment uses pure "Consumers" and "Providers" for simplicity reasons.

### The Consumer

A "Consumer" is an organization that aquires access to the Dataservices of a "Provider", in our Dataspace through the Marketplace. 
Since the "Provider" in our Dataspace uses a DSAB-compliant Auth-Framework, the "Consumer" can issue [VerifiableCredentials](https://www.w3.org/TR/vc-data-model/) to its users or employess, so that they can access the "Providers" services. 

![consumer](docs/consumer.png)

The consumer has the following components:

- Keycloak as the IDM system, providing the capabilities to issue VerifiableCredential to users and also [Gaia-X Self-Descriptions](https://gaia-x.gitlab.io/technical-committee/architecture-document//operating_model/#self-descriptions-and-attestation) to its Legal-Representatives. 
- Walt-Id to create and sign the actual Credentials and offer the did.json, corresponding to the participants identity


### The Provider

"Provider" offer various kinds of Dataservices throught the Marketplace and make them accessible through the IAM-Framework. In our Dataspace, the "Provider" Packet Delivery Co. provides a Portal, where users can follow the delivery of there orders and, depending on the level of service, change delivery details like the planned time of arrival. Information about the delivery is stored in the Orion-LD ContextBroker, thus can be secured by a DSBA-compliant IAM-Framework. 
Since the IAM-Framework is the same as the "Dataspace Operator" uses for the OnBoarding, the architecture looks quite similar:

![provider](docs/provider.png)

The components:

- Keycloak to provide credentials for employees, in the same way as for the [consumer](#the-consumer), to allow self-registration by the provider
- Walt-Id to create and sign credentials and offer the did.json, as the [consumer](#the-consumer) does, and also verify variuous parts of the credential(f.e. the signature) like the [dataspace operator](#the-dataspace-operator)
- the Activation Service, providing an interface to the Marketplace to get new "Consumers" registered in the Trusted Issuers List
- the Portal as GUI for the Users to log-in and see/update their deliveries
- the Verifier to provide the authentication capabilites for the portal(allowing users to log-in) and the Activation Service(allowing the Marketplace to access via M2M-flow)
- the Credentials Config Service, providing information about the scope to be requested and its trust-anchors, same as for the [dataspace operator](#the-dataspace-operator)
- the Trusted Issuers List, used by the Verifier to check the capabilities of an issuer and the Activation Service to add new "Consumer" as trusted issuers
- Kong, to enforce authorization on requests towards Orion-LD(same as [dataspace operator](#the-dataspace-operator))
- the PDP to decide on requests towards Orion-LD(same as [dataspace operator](#the-dataspace-operator))
- the Authorization Registry to provide policies (same as [dataspace operator](#the-dataspace-operator))
- Orion-LD to store and provide information about the deliveries

Additionally, the Verifier connects to the Trusted Participants API of the [Dataspace Operator](#the-dataspace-operator) to verify users as participants of the dataspace.


## The Marketplace