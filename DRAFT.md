# Electronic Permit

## Introduction

Permit is a document that authorizes the vehicle in a transport operation to cross the border of the destination country. In the current system, this document (permit) is physically prepared/printed by destination country and delivered to the corresponding country in a certain number. Then the haulier country issues a permit credential for vehicle. Difficulties in the implementation of this system and the security issues required the digitalization of the permit.
This draft text represents the arrangement of the specifications about the e-permit (digital permit).

## Problem and Motivation

Electronic permit can be considered as a common credential except some of its features. 

- It is exchanged in a certain number (quota) between two countries through protocols.
- The parties that deliver and receive the e-permit are known in advance of the exchange.
- Information about the usage (number of used/unused permits) of the permit is important for the countries (permit/quota management).

Considering the above-mentioned characteristics; the e-permit has an authentic format, it is created by the issuing authority in an electronically portable way and signed electronically to be communicated through a safe channel to the verifier country. In this scenario, e-permit - with a portable/mobile signature- fulfills the safety conditions such as identity validation, integrity and undeniability. Besides all these, in the event of a connection problem, it ensures offline verification as well. To sum up, the e-permit should have the below features: 

- Decentralized Trust
- Extensible and easy integrate (HTTP)
- Secure(Non Repudiation, Authentication, Integrity)
- Online/Offline Verification

![w:1000](img/e-permit-flow.png)


## Implementation

### Identity Management

Electronic identity management is needed in order to provide safe and undeniable (with digital signature) electronic communcition between two countries. In such scenarios where the parties are certainly known, the identity management can be performed with the below method easily.
- Each country produces its own key pair.
- Each country keeps the private key safely in its system for signing e-permit
- The public key is sent to the other country for the verification of signature. 

### Quota Management

### Issuing Permit

The issuing country prepares/forms the credentials of the document in accordance with the e-permit format and signs it with its private key. Then, the country sends the portable electronic document to the corresponding country through web service. The newly formed document is also sent to the carrier for offline verification.

### Permit Verification

As the e-permit, under normal circumstances,  will be sent to the other party through web service, the verification will be done online as default. However, it can be realised offline in cases of network problems.

#### Online Verification

The information of the newly formed e-permit can be displayed and the e-signature can be verified as the verifier authority enters the single permit identifier.

### Claims of e-permit credential(through web services)

| Code | Field | Description | Required | Format | Sample Value | 
| ---- | ------| ----------- | -------- | ------ | ------------ | 
| 1 | permit_year | Year of the permit | &#9745; | Year | 2020 |
| 2 | expire_at |  Permit valid until | &#9745; | dd/MM/yyyy | 31/01/2022 |
| 3 | serial_number | Serial Number of the permit | &#9745; | Number | 1 |
| 4 | issuer | This permit issued by |  &#9745; | Country code | ua |
| 5 | issued_at | This permit prepared on | &#9745; | dd/MM/yyyy | 01/03/2021 |
| 6 | company_name | Name of the company |&#9745; | Text(max 100) | Sample Org. |
| 8 | plate_number | Plate number(s) | &#9745; | Text | 06BB2020 |
| 9 | permit_type | Type of the permit | &#9745; | Enum[1,2,3] | "biliteral", "transit", "3rdcountry" |
| 11 | issued_for | This permit issued for | &#9745; | Country code | tr |
| 12 | claims | Other claims | &#9745; | Key Value | {} |

#### Offline Verification

In case of a network problem, the e-permit that is given to the carrier as qr code, can be verified offline by using the “Verifier Application. Countries can either develop their own verifier applications or use the universal verifier web application.( Universal Verifier Application (https://e-permit.github.io/verify)
Format of the permit is created by the addition of version information to the JWS format:
```{version}.{header}.{payload}.{signature}```(version: 0 means demo)

### Claims of e-permit offline credential

| Code | Field | Description | Required | Format | Sample Value | 
| ---- | ------| ----------- | -------- | ------ | ------------ | 
| 1 | exp |  Permit valid until | &#9745; | dd/MM/yyyy | 31/01/2022 |
| 2 | id | Permit identifier | &#9745; | ISSUER-VERIFIER-YEAR-TYPE-SERIALNUMBER | TR-UZ-2021-1-1 |
| 3 | iat | This permit prepared on | &#9745; | dd/MM/yyyy | 10/01/2021 |
| 4 | cn | Name of the company | | Text(max 100) | Sample Org. |
| 5 | pn | Plate number(s) | &#9745; | Text | 06AA1234 |


## Events

All communication between parties 

### Event Format 

An e-permit event can be considered as a standardJWS format, apart from the version information. JWS format is shown below:

![w:1000](https://raw.githubusercontent.com/e-permit/e-permit.github.io/master/img/jws-format.png)

### KEY_CREATED

- jwk: jwk content(kid, x, y, ...)

### KEY_REVOKED

- key_id: Id of the key(kid)
- revoked_at: Revocation timestamp

### QUOTA_CREATED

- permit_year:
- permit_tpe:
- start_number: 
- end_number:
### PERMIT_CREATED

- permit_id
- permit_year
- permit_type
- serial_number
- issued_at
- expire_at
- company_name
- plate_number
- claims

### PERMIT_REVOKED

- permit_id

### PERMIT_USED

- permit_id:
- activity_type:
- activity_timestamp
