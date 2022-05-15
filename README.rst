#######################################################
Trust Framework: trusted registration of legal entities
#######################################################

These are the Smart Contracts implementing the Trust Framework, which is composed of two things:

1. A list of the identities of trusted organisations stored in the blockchain, together with associated information for each entity.
2. A process to add, modify and delete the trusted entities.

The trust framework is designed to be largely decentralised and represents the trust relationships in the real world.

The identities of the juridical persons involved in the ecosystem are registered in a common directory implemented in the blockchain following a hierarchical scheme very similar to the DNS (Domain Name Service) schema in the Internet. Once an entity is registered in the system, it is completely autonomous for adding other entities that are managed as child entities.

However, there is one centralised element: the root of trust at the top of the hierarchy should be a trusted entity in the ecosystem that is the one bootstraping the system. Typically it should be a regulatory body or a public administration.

The approach is described in the following figure.

.. figure:: images/TrustFramework.png
   :width: 80 %
   :alt: The Trust Framework in the blockchain

   The Trust Framework in the blockchain.


The Trust Framework in a given blockchain is not really a flat list, but a hierarchical structure, implemented as a Smart Contract:

* There is a special organisation which is at the root of the hierarchy. Ideally, this is a regulator, like the Central Bank of the country to manage banks, or the Ministry of Education to manage universities.
* This root entity is responsible for registering the identities of some trusted entities. For example, in a country with several regions with autonomous competencies to manage universities, the Ministry of Education could register in the blockchain the identities of the regional institutions which are responsible for managing the universities in each of their regions.
* Once this is done, each of the regional institutions can register the identities of dependent entities, like universities.
* The hierarchy can have several levels. For example, a university can be big and have several organisational units with some autonomy, maybe distributed geographically. It can create sub-identities and register them as child nodes in the blockchain.

Some observations about this structure:

* An organisation can be registered in the blockchain only because its parent entity has registered it. No other entity in the Trust Framework can have performed the registration, not even the parent of the parent entity.
* An organisation is responsible for all its child entities, represented as child nodes in the blockchain.
* A third party external to the framework 


**********************************
Description of the Smart Contracts
**********************************

The Trust Framework is composed of the following:

The main registry
*****************

ENSRegistry.sol
    Implements the main registry and is the first contract that should be deployed.

ENS.sol
    Defines the interfaces and is included by ``ENSRegistry`` and other contracts.


The resolver contracts
**********************

These contracts implement resolution of different types of information that can be registered in the blockchain.

PublicResolver.sol
    This is the main contract and the only one that must be compiled, because all others are included.

ResolverBase.sol
    A utility contract used by all other contracts specialised in resolving different types of information.

Inside the ''profiles'' directory we can find the specialised contracts, all of them are included by the
``PublicResolver.sol`` contract.

AlaDIDPublicEntityResolver.sol
    Registers and resolves the identities of juridical persons (businesses, institutions, etc.). It keeps the DID
    and information required to build the associated DID Document. This contract registers and resolves the first
    public key associated to the DID. Additional public keys are registed using the ``AlaDIDPubkeyResolver`` contract.

AlaDIDPubkeyResolver.sol
    Registers and resolves the public keys associated to the DID. In reality, it is used only if the DID has more
    than one public key. In the simple case where the DID has only one public key, it is registered and resolved
    directly by the ``AlaDIDPublicEntityResolver`` contract.

AlaPublicCredentialResolver.sol
    Registers and resolves metadata about public credentials associated to the DID. In the case of juridical persons,
    they may want to register publicly available credentials (e.g. self-declarations). In general, the bulk of the
    credential data should be stored off-chain, but this contract allows registering critical credential metadata,
    including a tamper-resistant pointer to the actual location of the credential details.

AlaTSPResolver.sol
    Registers and resolves special entities. It is a specialisation of ``AlaDIDPublicEntityResolver`` in the sense
    that it is intended for registration of the Trust Service Providers (TSPs), implementing the EU TSP List of Lists
    on the blockchain. It contains the associated X509 certificates for each TSP, making it very efficient to verify
    digital signatures existing off-chain.

NameResolver.sol
    It performs reverse resolution, from domain name to blockchain address. Each entity in the Trust Framework has
    a domain name assigned at the moment of registration, much in the same way as it happens with standard domain
    names. This contract provides a secure way to determine from the domain name the blockchain address associated
    to it and then retrieve all additional information.

ContentHashResolver.sol
    Registers and resolves Content-addressable Identifiers. It provides the ability for a DID to register pointers
    to additional information that they want to make available to the public.

TextResolver.sol
    A general registration and resolution of any text that the DID wants to store associated to its DID. The content
    is free-format and can be used for almost any purpose that the DID wishes. However, being free-format means that
    the actual format and semantics may not be understood by everybody.


*********************************
Deployment of the Smart Contracts
*********************************

Compilation
***********

Only two contract have to be compiled: ``ENSRegistry.sol`` and ``PublicResolver.sol``. All the others are included
by these two.

Deployment
**********

The deployment has to be performed in strict order: first ``ENSRegistry.sol`` and then ``PublicResolver.sol``.

The constructor of ``PublicResolver.sol`` requires the deployed address of ``ENSRegistry.sol``, so it can invoke its
functions when registering/resolving information.
