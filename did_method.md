# OmniOne DID Method Specification

## Abstract

 OmniOne is a decentralized network system for  Self-Sovereign identity and Verifiable Claims.
It can replace a legacy centralized credential system that with trusted blockchain node.
In the OmniOne system, several types of certificates are issued. DID(Decentralized Identifiers) is used as the unique identifier of the certificate.
Also DID allows to obtain public key information for secure exchange of information between users.

## Status of This Document

This document is not a W3C Standard. It is a draft document and will be updated.

## OmniOne DID Method Name

The name string that shall identify this DID method is: omn


## OmniOne DID Format

ALPHA and DIGIT used in "omn-identifier" are the definition of Core Rules in [RFC5234].

```
omn-did        = "did:omn:" omn-identifier
omn-identifier = ALPHA / DIGIT
```

```
Example
did:omn:4EFNaYeA9hDp6F55JAB38EFtNcYEbbM9nwKr
```



## Operation
OmniOne DID (CRUD) operations are provided in RESTful API format.
DID documents are generated in the block chain by smart contract code.
it checks the authority and performs signature verification within the smart contract.
To prevent replay attacks, all signatures require a nonce, which is stored in the blockchain.

### Create

Two types of authoring are supported.
You can request  a DID generation through simple input data and signature values.
The other is to create and send the entire DID Doucment directly.
SmartContract returns true if it has not been issued to a block chain and if the signature verification succeeds.

To create a simple type of document, the input value must be as following.
It returns "true", if successfully created.

```
endpoint : /did_create_simple
input : {did , publicKey , signature}
output : {result}
```

example input data 
```
{  
   "id":"did:omn:7V2FnzCykod7aK9eMBEtKEdyfxSwn",
   "publicKey":[  
      "677AgijZXPuwmhSVMTkNXGArgMY7GA9iyhrMj8gs",
      "3DzeDRdey97pWydGAuKGEWZKpBzjevwGB4NbyZPkkVs4RoF"
   ],
   "signature":"5nbzWuDGXyb...FPg6HmZ4JM6q=="
}
```
example output data	
```
{
"result": true,
"message" : "did:omn:7V2FnzCykod7aK9eMBEtKEdyfxSwn created"
}
```

To create a full type of document, the input value must be as following.
The input has full docuemnt of did with signature.
It returns "true", if successfully created.
```
endport : /did_create
input : {did_document}
output : {result}
```

example input data
```
{  
   "id":"did:omn:7V2FnzCykod7aK9eMBEtKEdyfxSwn",
   "publicKey":[  
      {  
         "id":"did: omn:7V2FnzCykod7aK9eMBEtKEdyfxSwn #keys-1",
         "type":"EcdsaKoblitzSignature2016",
         "publicKeyPem":"-----BEGIN PUBLIC KEY...END PUBLIC KEY-----\r\n"
      },
      {  
         "id":"did:omn:7V2FnzCykod7aK9eMBEtKEdyfxSwn#keys-2",
         "type":"EcdsaKoblitzSignature2016",
         "publicKeyPem":"-----BEGIN PUBLIC KEY...END PUBLIC KEY-----\r\n"
      }
   ],
   "authentication":[  
      {  
         "publicKey":"did:omn:7V2FnzCykod7aK9eMBEtKEdyfxSwn#keys-1"
      }
   ],
   "proof":{  
      "type":" EcdsaKoblitzSignature2016",
      "created":"2018-08-02T16:01:10Z",
      "nonce":"dQew214232",
      "creator":"did:omn:7V2FnzCykod7aK9eMBEtKEdyfxSwn#keys-1",
      "signatureValue":"Ee2.....=="
   }
}
```

example output data	
```
{
"result": true,
"message" : "did:omn:7V2FnzCykod7aK9eMBEtKEdyfxSwn created"
}
```

### Read 

To read did document for some did, the input value must be as following.
It returns a did docuement for it, if it is found.

```
endport : /did_read
input : { did }
output : { did_document }
```

```
example input data
{
"id": "did:omn:7V2FnzCykod7aK9eMBEtKEdyfxSwn"
}
```

```
example output data
{  
   "id":"did:omn:7V2FnzCykod7aK9eMBEtKEdyfxSwn",
   "publicKey":[  
      {  
         "id":"did: omn:7V2FnzCykod7aK9eMBEtKEdyfxSwn #keys-1",
         "type":"EcdsaKoblitzSignature2016",
         "publicKeyPem":"-----BEGIN PUBLIC KEY...END PUBLIC KEY-----\r\n"
      },
      {  
         "id":"did:omn:7V2FnzCykod7aK9eMBEtKEdyfxSwn#keys-2",
         "type":"EcdsaKoblitzSignature2016",
         "publicKeyPem":"-----BEGIN PUBLIC KEY...END PUBLIC KEY-----\r\n"
      }
   ],
   "authentication":[  
      {  
         "publicKey":"did:omn:7V2FnzCykod7aK9eMBEtKEdyfxSwn#keys-1"
      }
   ]
}
```

### Update
To update did_document ,the input value must be as following.
The input has a new document with signature including a nonce.
It returns updated a did docuemnt, if successfully updated.

```
endport : /did_update
input : did_document (included proof)
output : did_document
```

example input data

```
{  
   "id":"did:omn:7V2FnzCykod7aK9eMBEtKEdyfxSwn",
   "publicKey":[  
      {  
         "id":"did: omn:7V2FnzCykod7aK9eMBEtKEdyfxSwn #keys-1",
         "type":"EcdsaKoblitzSignature2016",
         "publicKeyPem":"-----BEGIN PUBLIC KEY...END PUBLIC KEY-----\r\n"
      },
      {  
         "id":"did:omn:7V2FnzCykod7aK9eMBEtKEdyfxSwn#keys-2",
         "type":"EcdsaKoblitzSignature2016",
         "publicKeyPem":"-----BEGIN PUBLIC KEY...END PUBLIC KEY-----\r\n"
      },
      {  
         "id":"did:omn:7V2FnzCykod7aK9eMBEtKEdyfxSwn#keys-3",
         "type":"EcdsaKoblitzSignature2016",
         "publicKeyPem":"-----BEGIN PUBLIC KEY...END PUBLIC KEY-----\r\n"
      }
   ],
   "authentication":[  
      {  
         "publicKey":"did:omn:7V2FnzCykod7aK9eMBEtKEdyfxSwn#keys-1"
      }
   ],
   "proof":{  
      "type":" EcdsaKoblitzSignature2016",
      "created":"2018-08-02T16:01:10Z",
      "nonce":"dQew214232",
      "creator":"did:omn:7V2FnzCykod7aK9eMBEtKEdyfxSwn#keys-1",
      "signatureValue":"Ee2.....=="
   }
}
```

example output data

```
{  
   "id":"did:omn:7V2FnzCykod7aK9eMBEtKEdyfxSwn",
   "publicKey":[  
      {  
         "id":"did: omn:7V2FnzCykod7aK9eMBEtKEdyfxSwn #keys-1",
         "type":"EcdsaKoblitzSignature2016",
         "publicKeyPem":"-----BEGIN PUBLIC KEY...END PUBLIC KEY-----\r\n"
      },
      {  
         "id":"did:omn:7V2FnzCykod7aK9eMBEtKEdyfxSwn#keys-2",
         "type":"EcdsaKoblitzSignature2016",
         "publicKeyPem":"-----BEGIN PUBLIC KEY...END PUBLIC KEY-----\r\n"
      },
      {  
         "id":"did:omn:7V2FnzCykod7aK9eMBEtKEdyfxSwn#keys-3",
         "type":"EcdsaKoblitzSignature2016",
         "publicKeyPem":"-----BEGIN PUBLIC KEY...END PUBLIC KEY-----\r\n"
      }
   ],
   "authentication":[  
      {  
         "publicKey":"did:omn:7V2FnzCykod7aK9eMBEtKEdyfxSwn#keys-1"
      }
   ]
}
```

### Delete
To delete did_document, the input value must be as following.
The input has a signature including a nonce. 

```
endport : /did_delete
input : did with proof
output : result
```

After blockchain verify the proof of the delete operation with the public key, 
the did_document(include public keys) is deleted from the blockchain and the
did can no longer be used. And this operation can not be undone.

It returns "true", if successfully delete.


example input data

```json
{  
"id":"did:omn:7V2FnzCykod7aK9eMBEtKEdyfxSwn",
"proof":{  
"type":" EcdsaKoblitzSignature2016",
"created":"2018-08-02T16:01:10Z",
"nonce":"d235qsd",
"creator":"did:omn:7V2FnzCykod7aK9eMBEtKEdyfxSwn#keys-1",
"signatureValue":"d976.....=="
}
}
```

example output data

```
{  
"result": true,
"message": "did:omn:7V2FnzCykod7aK9eMBEtKEdyfxSwn deleted"
}
```


# Security Considerations
### Replay Attacks 
In order to prevent a replay attact a did proof has to have random nonce value. 
In decentralized network or smart contract code could not generate reliable random number.(oracle problem)
To solve this issue , we store the nonce value that was used , and check if the value has already been used .

### Non-repudiation
The DID owner generates proof(digital  signature) with the private key and the signature is verified with a public key that is paired with the private key in the blockchain. With this process non-repudiation is satisfied.

### Providing Traffic Security
The OmniOne Network uses TLS protocol to provide traffic security.

### Storing Data
Currently, OmniOne only stores the following data on the blockchain.
- DID Document which includes public keys
- signature values, which are used to verify the proof of the verifiable claim

### Consensus Algorithm
In order to ensure the stability of blockchain nodes, OmniOne uses a signature-based Delegated Proof of Stake consensus algorithm and BFT (2/3 or more agreements)

# Privacy Considerations
OmniOne blockchain and DID Documents contain no PII(Personally-Identifiable Information).
Currently OmniOne  only stores the following data on the blockchain.
- DID Documents which includes public keys
- signature values which are used to verify the proof of the verifiable claim

# References
[1]. W3C Decentralized Identifiers (DIDs) v0.11, https://w3c-ccg.github.io/did-spec/.
