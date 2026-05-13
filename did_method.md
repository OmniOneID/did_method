# OmniOne DID Method Specification

## Abstract

OmniOne is a decentralized network system for Self-Sovereign Identity and Verifiable Credentials.
It can replace a legacy centralized credential system with a trusted blockchain node.
In the OmniOne system, several types of certificates are issued. DID (Decentralized Identifiers) is used as the unique identifier of the certificate.
Also DID allows to obtain public key information for secure exchange of information between users.

## Status of This Document

This document is a specification for the `did:omn` method, registered in the [W3C DID Specification Registries](https://www.w3.org/TR/did-extensions-methods/).

## OmniOne DID Method Name

The name string that shall identify this DID method is: `omn`

## OmniOne DID Format

The OmniOne DID scheme is defined by the following ABNF.
ALPHA and DIGIT are the definition of Core Rules in [RFC5234].

```abnf
omn-did            = "did:omn:" omn-specific-id
omn-specific-id    = *( omn-subsegment ":" ) omn-identifier
omn-subsegment     = 1*( ALPHA / DIGIT / "-" / "_" )
omn-identifier     = 1*( ALPHA / DIGIT )
```

The `method-specific-id` format MAY include colons as hierarchical namespace delimiters, as permitted by [W3C DID Core Section 8.1](https://www.w3.org/TR/did-core/#method-syntax). Zero or more `omn-subsegment` are allowed before the final `omn-identifier`.

The method name `omn` MUST be lowercase. The `omn-specific-id` is case-sensitive.

```
Examples:
did:omn:4EFNaYeA9hDp6F55JAB38EFtNcYEbbM9nwKr
did:omn:opendid:mainnet:issuer
```

## Operation

DID documents are managed on the OmniOne Chain (Hyperledger Besu) by smart contract code.
The smart contract verifies authorization before executing any operation.

### Create

The DID controller registers a new DID Document on the OmniOne Chain through the smart contract.
The smart contract verifies that the DID does not already exist on the chain and stores the DID Document upon successful verification.

```
function : registDidDoc
input : {InvokedDidDoc, RoleType}
output : void
```

example input data (abbreviated; `...` denotes elided fields)
```
{
  "didDoc": {
    "didDoc": "z3R...MultiBase-encoded DID Document",
    "controller": {
      "did": "did:omn:opendid:mainnet:tas",
      ...
    },
    "proof": {
      "verificationMethod": "did:omn:opendid:mainnet:tas?versionId=1#key1",
      ...
    },
    "nonce": "zW3d5...random-nonce"
  },
  "roleType": "ISSUER"
}
```

`RoleType`: A string that identifies the role of the DID controller within the OmniOne ecosystem (e.g., TAS, ISSUER, VERIFIER, WALLET_PROVIDER, etc.). The full set of role types is defined in the OmniOne specification.

### Read (Resolve)

A DID Document is retrieved from the OmniOne Chain by submitting the corresponding DID.
The authenticity of the response is guaranteed by the integrity of the underlying blockchain consensus.

The input is a single `didKeyUrl` string. A specific version MAY be requested by appending the `versionId` query parameter (`?versionId=N`); if omitted, the latest version is returned.

```
function : getDidDoc
input : didKeyUrl
output : {DidDocAndStatus}
```

example input data (latest version)
```
"did:omn:opendid:mainnet:issuer"
```

example input data (specific version)
```
"did:omn:opendid:mainnet:issuer?versionId=1"
```

example output data (abbreviated; `...` denotes elided fields)
```
{
  "document": {
    "@context": ["https://www.w3.org/ns/did/v1"],
    "id": "did:omn:opendid:mainnet:issuer",
    "controller": "did:omn:opendid:mainnet:tas",
    ...
    "verificationMethod": [ ... ],
    "assertionMethod": [ ... ],
    "authentication": [ ... ],
    "keyAgreement": [],
    ...
    "service": [ ... ]
  },
  "status": "ACTIVATED"
}
```

`DidDocStatus`: `ACTIVATED`, `DEACTIVATED`, `REVOKED`, `TERMINATED`

### Update

The DID controller submits a status update to the smart contract.
The DID MUST exist on the chain and MUST NOT already be in a terminal state (`REVOKED` or `TERMINATED`).
Two contract functions are used depending on the type of status transition.

To transition between `ACTIVATED` and `DEACTIVATED` states, the in-service function is used. This operation is reversible.

```
function : updateDidDocStatusInService
input : {did, DidDocStatus}
output : {DidDocument}
```

example input data
```json
{
  "did": "did:omn:opendid:mainnet:issuer",
  "didDocStatus": "DEACTIVATED"
}
```

example output data (abbreviated; `...` denotes elided fields)
```
{
  "@context": ["https://www.w3.org/ns/did/v1"],
  "id": "did:omn:opendid:mainnet:issuer",
  "controller": "did:omn:opendid:mainnet:tas",
  ...
  "deactivated": true,
  "verificationMethod": [ ... ],
  "assertionMethod": [ ... ],
  "authentication": [ ... ],
  "keyAgreement": [],
  ...
  "service": [ ... ]
}
```

To transition to `REVOKED` or `TERMINATED` state, the revocation function is used. These operations are irreversible.

```
function : updateDidDocStatusRevocation
input : {did, DidDocStatus}
output : {DidDocument}
```

example input data
```json
{
  "did": "did:omn:opendid:mainnet:issuer",
  "didDocStatus": "REVOKED"
}
```

### Deactivate

Deactivation in OmniOne is performed via the [Update](#update) operation with `DidDocStatus = DEACTIVATED`.

See [Update](#update) for the function, input, and output specification.

## Security Considerations

This section addresses the security requirements specified in [RFC3552](https://www.rfc-editor.org/rfc/rfc3552).

### Eavesdropping
The OmniOne Network uses TLS protocol to provide traffic security.
DID Documents contain only public keys and service endpoints — no private keys or personal information are transmitted or stored.

### Replay Attacks
The OmniOne Chain employs measures to prevent replay attacks on DID operations.

### Message Insertion, Deletion, and Modification
All write operations (create, update, deactivate) require authorization from the DID controller. The blockchain's immutable ledger and consensus mechanism prevent unauthorized insertion, deletion, or modification of DID Documents.

### Man-in-the-Middle
TLS protects the transport layer between clients and nodes. On-chain operations are protected by cryptographic authorization — unauthorized parties cannot execute DID operations without valid credentials.

### Amplification
The permissioned nature of the OmniOne Chain limits the risk of amplification attacks. Only authorized nodes participate in the network.

### Denial of Service
The OmniOne Chain uses a permissioned public network model based on Hyperledger Besu. Node participation is controlled through permissioning, which limits the attack surface for denial of service.

### Residual Risks
Smart contract vulnerabilities could potentially affect the integrity of DID operations. The security of the system depends on the correct implementation of the smart contract and the underlying blockchain platform.

### Integrity Protection and Authentication
All operations that modify the state of a DID Document require proof of control from the DID controller. The specific authentication mechanisms are defined by the implementation.

### Uniqueness
DID uniqueness is enforced by the smart contract, which rejects creation of a DID that already exists on the blockchain.

### Storing Data
OmniOne stores DID-related data on the blockchain. No private keys or personal information are stored on-chain.

### Consensus Algorithm
The OmniOne Chain uses a BFT-based consensus algorithm to ensure the integrity and stability of blockchain nodes.

## Privacy Considerations

This section addresses the privacy requirements specified in [RFC6973](https://www.rfc-editor.org/rfc/rfc6973). OmniOne blockchain and DID Documents contain no PII (Personally Identifiable Information).

### Surveillance
DID Documents stored on the blockchain are publicly readable. However, they contain only public keys and service endpoints — no personal data. Network-level surveillance is mitigated by TLS.

### Stored Data Compromise
OmniOne stores only DID-related data on the blockchain. No private keys or personal information are stored on-chain. Compromise of stored data does not expose private information.

### Unsolicited Traffic
The OmniOne Chain's permissioned model limits the ability of unauthorized parties to generate unsolicited traffic targeting DID subjects.

### Misattribution
DID operations require authorization from the DID controller, reducing the risk of actions being falsely attributed to a DID subject.

### Correlation
A DID is a persistent identifier that can be used to correlate activity across services. To mitigate correlation risks, DID subjects MAY use multiple DIDs for different contexts.

### Identification
OmniOne DIDs are pseudonymous. The DID itself does not reveal the real-world identity of the DID subject. However, information obtained through service endpoints or verifiable credentials linked to the DID may enable identification.

### Secondary Use
DID Documents are public on the blockchain. Third parties may use DID Document data beyond the DID subject's original intent.

### Disclosure
DID subjects should only include information in their DID Documents that they are willing to make public.

### Exclusion
DID subjects maintain control over their DID Documents through the update and deactivate operations, ensuring they are not excluded from managing their own identity data.

## References

- [W3C Decentralized Identifiers (DIDs) v1.0](https://www.w3.org/TR/did-core/)
- [W3C DID Specification Registries](https://www.w3.org/TR/did-spec-registries/)
- [RFC3552 - Guidelines for Writing RFC Text on Security Considerations](https://www.rfc-editor.org/rfc/rfc3552)
- [RFC5234 - Augmented BNF for Syntax Specifications: ABNF](https://www.rfc-editor.org/rfc/rfc5234)
- [RFC6973 - Privacy Considerations for Internet Protocols](https://www.rfc-editor.org/rfc/rfc6973)
- [Hyperledger Besu](https://besu.hyperledger.org/)
