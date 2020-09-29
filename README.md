# Electronic Permit

## Introduction

Permit is a document that authorizes the vehicle in a transport operation to cross the border of the destination country. In the current system, this document (permit) is physically prepared/printed and delivered to the corresponding country in a certain number. Difficulties in the implementation of this system and the security issues required the digitalization of the permit.
This draft text represents the arrangement of the specifications about the e-permit (digital permit). It has been inspired from the specifications such as JWS, JWT, OPENID CONNECT, VCs and DIDs.
## Problem and the Proposed Solution

Electronic permit can be considered as common credential except some of its new features. There is a specification study regarding the electronic management of a credential, which is called "Verifiable Credentials Data Model 1.0". Life cycle of the credential in the mentioned specificatoion is as follows:

![w:1000](https://miro.medium.com/max/619/0*QLb4tYr-R5foQkXT)

During the adaptation of some of the concepts in the picture above, “issuer” and “verifier” can be named as the country, “subject” as the vehicle plate number and “holder” as the driver. However, with its features below, e-permit differs from the said life cycle :
    • It is exchanged in a certain number (quota) between two countries through Protocols.
    • The parties that deliver and receive the e-permit are known in advance of the exchange.
    • Information about the usage (number of used/unused permits) of the permit is important for the countries (situation management).
Considering the above-mentioned characteristics; the e-permit has an authentic format, it is created by the issuing authority in an electronically portable way and signed electronically to be communicated through a safe channel to the verifier country. In this scenario, e-permit - with a portable/mobile signature- fulfills the safety conditions such as identity validation, integrity and undeniability. Besides all these, in the event of a connection problem, it ensures offline verification as well. To sum up, the e-permit should have the below features: 

- Decentralized (P2P) Trust(Certificate Authority or Distributed Ledger is not required )
- Extensible and easy integrate (Rest http api)
- Secure(Non Repudiation, Authentication, Integrity)
- Online (document is sent through web service) / Offline (portable document with digital signature) Verification

## Implementation

### Identity Management

Electronic identity management is needed in order to provide safe and undeniable (with digital signature) electronic communcition between two countries. In such scenarios where the parties are certainly known, the identity management can be performed with the below method easily.
- Each country produces its own key pair.
- Each country keeps the private key safely in its system for signing e-permit
- The public key is sent to the other country for the verification of signature. Key sharing may be realized in written and original signed document during the arrangement of the Protocol between the countries.

### Issuance

The issuing country prepares/forms the credentials of the document in accordance with the e-permit format and signs it with its private key. Then, the country sends the portable electronic document to the corresponding country through web service. The newly formed document is also sent to the carrier for offline verification.

### Verification

As the e-permit, under normal circumstances,  will be sent to the other party through web service, the verification will be done online as default. However, it can be realised offline in cases of network problems.

#### Online Verification

The information of the newly formed e-permit can be displayed and the e-signature can be verified as the verifier authority enters the single number (hash of the permit).

#### Offline Verification

In case of a network problem, the e-permit that is given to the carrier as qr code, can be verified offline by using the “Verifier Application. Countries can either develop their own verifier applications or use the existing web system.( Universal Verifier Application (https://e-permit.github.io/verify)

### Flow

![w:1000](https://raw.githubusercontent.com/e-permit/e-permit.github.io/master/img/e-permit-flow.png)

### Claims of e-permit credential

| Code | Field | Description | Required | Format | Sample Value | 
| ---- | ------| ----------- | -------- | ------ | ------------ | 
| 1 | py | Year of the permit | &#9745; | Year | 2020 |
| 2 | exp |  Permit valid until | &#9745; | Unix Epoch Time | 1311281970 |
| 3 | pid | Serial Number of the permit | &#9745; | Number | 1 |
| 4 | iss | This permit issued by |  &#9745; | Country code | ua |
| 5 | iat | This permit prepared on | &#9745; | Unix Epoch Time | 1311281970 |
| 6 | cn | Name of the company | | Text(max 100) | Sample Org. |
| 7 | cid | National ID of the company | &#9745; | Text | A101.. | 
| 8 | sub | Plate number(s) | &#9745; | Text | 06BB2020 |
| 9 | pt | Type of the permit | &#9745; | Enum[1,2,3] | "biliteral", "transit", "3rdcountry" |
| 10 | res | Restrictions | | Text(max 100) | Sample res. |
| 11 | aud | This permit issued for | &#9745; | Country code | tr |



### Credential Format 

An e-permit can be considered as a standardJWS format, apart from the version information. JWS format is shown below:

![w:1000](https://raw.githubusercontent.com/e-permit/e-permit.github.io/master/img/jws-format.png)

Format of the permit is created by the addition of version information to the JWS format:
```{version}.{header}.{payload}.{signature}```


### Verifier Services

- ```/credentials```: Issuer country sends e-permit credential by using the service(Post Method)
- ```/credentials/{id}/revoke```: Issuer country sends e-permit credential revocation by using the service(Post Method)


### Issuer Services

- ```/credentials/{id}```: Verifier country gets e-permit credential by using the service(Get Method). 
- ```/credentials/{id}/used```: Verifier country sends usage status to issuer  by using the service(Post Method).

## Demo

- [Issuer Demo]( https://e-permit.github.io/demo/)
- [Universal Verifier Application(camera is required)]( https://e-permit.github.io/verify/)






