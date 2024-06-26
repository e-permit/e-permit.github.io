# Electronic Permit

## Introduction

Permit is a document that authorizes the vehicle in a transport operation to cross the border of the destination country. In the current system, this document (permit) is physically prepared/printed by destination country and delivered to the corresponding country in a certain number. Then the haulier country issues a permit credential for vehicle. Difficulties in the implementation of this system and the security issues required the digitalization of the permit.
This draft text represents the arrangement of the specifications about the e-permit (digital permit).

## Problem and Solution

Permit can be considered as a common credential except some of its features. 

- It is exchanged in a certain number (quota) between two countries through protocols.
- The parties that deliver and receive the e-permit are known in advance of the exchange.
- Information about the usage (number of used/unused permits) of the permit is important for the countries (permit quota/usage).

![w:1000](img/e-permit-flow.png)

Considering the above-mentioned characteristics; the permit has an authentic format, it is created by the issuing authority and signed for the verifier country in a certian number. In this scenario, e-permit solution uses a standard based on electronic data exchange with sealed data. According to this standard, each country must provide its own e-seal and web service for data exchange. The provided web service must be a REST API. A country must seal the message it wants to send with its own private key and send it through the service provided by the other country. The process begins with the mutual identification of the countries. Each country must share its e-seal public keys with the service provided. Once the countries have mutually identified each other, they will be able to exchange data. A sample scenario will be more illustrative for the following process.

For example, let's consider the issuance of e-permits and sharing of entry-exit information for transport companies operating between Turkey and Uzbekistan. In this scenario, first, Uzbekistan needs to set quotas for Turkish transport companies. This process is detailed here. Turkey can then issue permits to its transport companies using the set quota. The permit information is sent to Uzbekistan using this service method. Turkey sends the transport company the permit document number and QR code. The transport company can then enter and exit Uzbekistan with the permit. These entries and exits are sent to the Turkish system by Uzbekistan using this method.


## Implementation

### Events

When a country defines a quota, creates / cancels a permit or when a counterpart’s transport operator enters the country, these data will have to be sent to the counterpart. These data are named as “event”. Each country which is integrated electronically should be able to create and send these events to the counterpart and process the events created by the counterpart.  

Each event should contain certain fields. These fields are shown in the following table:

| No | Field | Description | Format | 
| ---- | ------| ----------- | -------- | 
| 1 | event_id |  The unique identifier of event | Text(e.g. ABC123..) |
| 2 | previous_event_id | Previous event identifier | Text(e.g. ABC123..) |
| 3 | event_type | Event type | Enum[KEY_CREATED, KEY_REVOKED, QUOTA_CREATED, PERMIT_USED, PERMIT_CREATED, PERMIT_REVOKED] | 
| 4 | event_timestamp | The UTC time of the event | Long(1625304893) |
| 5 | event_publisher | Event publisher | Two letter country code(e.g. TR, UZ, UA) |
| 6 | event_consumer | Event consumer | Two letter country code(e.g. TR, UZ, UA)  |

When an event is created producer country should send this event with an authorization header(JWS). Consumer party should send a response like below:

- 401 for invalid jws
- 400 for bad request
- 200 for succeed 

```
{
  "ok": false,
  "error_code": "EVENT_ALREADY_EXISTS"
}
```

#### Error Codes

- EVENT_ALREADY_EXISTS,
- PREVIOUS_EVENT_NOTFOUND,
- GENESIS_EVENT_ALREADY_EXISTS,
- KEYID_ALREADY_EXISTS,
- KEY_NOTFOUND,
- INSUFFICIENT_KEY,
- INSUFFICIENT_PERMIT_QUOTA,
- INVALID_QUOTA_INTERVAL,
- PERMITID_ALREADY_EXISTS,
- PERMIT_NOTFOUND,
- INVALID_PERMITID;

#### Identity Management

Electronic identity management is needed in order to provide safe and undeniable (with digital signature) electronic communcition between two countries. In such scenarios where the parties are certainly known, the identity management can be performed with the below method easily.
- Each country produces its own key pair.
- Each country keeps the private key safely in its system for signing e-permit
- The public key is sent to the other country for the verification of signature.

#### ```/epermit-configuration``` ```GET```

When a country wants to add another country it uses this resource to retrieve information about the country. 
Sample response:

```
{
      "code": "TR",
      "name": "Turkey",
      "keys": [
        {
          "kty": "EC",
          "crv": "P-256",
          "x": "b-twdhMdnpLQJ_pQx8meWsvevCyD0sufkdgF9nIsX-U",
          "y": "U339OypYc4efK_xKJqnGSgWbLQ--47sCfpu-pJU2620",
          "use": "sig",
          "kid": "1",
          "alg": "ES256"
        }
      ]
 }
```

#### Generating Proof

Events should contain a proof for secure communication between countries. This proof should be sent in ```Authorization``` header with ```Bearer ``` prefix. When a country produces an event, it should generate a `JWS` for the event. A JWS contains 3 parts(header, payload, signature). Payload is the event content in e-permit scenario. So it is sufficient to use header and signature as a proof ```<header>.<signature>``` for events. The steps are like following:

- Create an event
- Generate a ```JWS``` with using event as payload
- Generate a proof with using ```<header>.<signature>```
- Add the proof in ```Authorization``` header with ```Bearer ``` prefix

#### Verifying Proof

When an event is received, verifier country should verify proof. Verifying can be implemented the following way:

- Get ```Authorization``` header from the request. Sample: ```Bearer <proof>```
- Omit the pure proof. 
- Combine proof with payload. ```<header>.base64URL(<json payload>).<signature>```
- Verify the generated ```JWS``` with the ```JOSE``` library

#### ```/events/key-created``` ```POST```

A country uses this resource when it creates a key. 

| No | Field | Description | Format | 
| ---- | ------| ----------- | -------- | 
| 1 | kid |  Key identifier | Text(e.g. 2) |
| 2 | kty | Key type | P-256 |
| 3 | use | Usage | sig | 
| 4 | crv | Curve | P-256 |
| 5 | x | Public key y value | b-twdhMdnpLQJ_pQx8meWsvevCyD0sufkdgF9nIsX-U |
| 6 | y | Public key x value | U339OypYc4efK_xKJqnGSgWbLQ--47sCfpu-pJU2620 |
| 7 | alg | The jws algorithm | ES256 |

#### ```/events/key-revoked``` ```POST```

A country uses this resource when it revokes a key. 

| No | Field | Description | Format | 
| ---- | ------| ----------- | -------- | 
| 1 | key_id |  Key identifier | Text(e.g. 2) |
| 2 | revoked_at | The UTC time of revocation | Long(123444) |

### Quota Management

A country, should define serial number quotas in certain intervals for counterpart’s transport operators. This process is defined as quota description. By this way counterpart can produce permits for its own hauliers in the defined quota interval. This event contains the following fields.

#### ```/events/quota-created``` ```POST```

A country uses this resource when it creates a permit. 

| No | Field | Description | Format | 
| ---- | ------| ----------- | -------- | 
| 1 | permit_issuer | Permit issuer for the quota | UZ |
| 1 | permit_issued_for | Permit issued_for for the quota | TR |
| 1 | permit_year |  Year of the quota | 2021 |
| 2 | permit_type | Permit type of the quota | BILITERAL |
| 3 | start_number | Start number of the quota | 20 |
| 4 | end_number | End number of the quota | 50 |

### Permit Management

#### ```/events/permit-created``` ```POST```

A country uses this resource when it creates a permit. 

| | Field Code | Description | Example | 
| ---- | ------| ----------- | -------- | 
| (1) | permit_id |  PERMIT SERIAL NUMBER | TR-UZ-2021-1-1 |
| (2) | permit_type | TYPE OF THE PERMIT | BILITERAL |
| (3) | permit_year |  YEAR OF THE PERMIT | 2021 |
| (4) | permit_issuer | PERMIT ISSUED BY | TR |
| (5) | permit_issued_for | PERMIT ISSUED FOR | UZ |
| (6) | plate_number |  PLATE NUMBER(S) | 06TEST1234 |
| (7) | issued_at |  PERMIT PREPARED ON | 03/03/2021 |
| (8) | expire_at |  PERMIT VALID UNTIL | 31/01/2022 |
| (9) | company_name |  NAME OF THE COMPANY | e.g. ABC Company |
| (10) | company_id |  ID OF THE COMPANY | e.g. 123 |
| (11) | departure_country |  DEPARTURE COUNTRY | e.g. TR |
| (12) | arrival_country |   ARRIVAL COUNTRY | e.g. TR |
| (13) | other_claims |  OTHER INFORMATION/RESTRICTIONS | ```{"res": "The permit is restricted..."}``` |

#### ```/events/permit-revoked``` ```POST```

A country uses this resource when it creates a permit. 

| No | Field | Description | Format | 
| ---- | ------| ----------- | -------- | 
| 1 | permit_id |  Permit identifier | TR-UZ-2021-1-1 |

#### ```/events/permit-used``` ```POST```

A country uses this resource when it creates a permit.


| No | Field | Description | Format | 
| ---- | ------| ----------- | -------- | 
| 1 | permit_id |  Permit identifier | TR-UZ-2021-1-1 |
| 2 | activity_type |  Usage type | ENTERANCE-EXIT |
| 3 | activity_timestamp | The UTC time of the activity  | Long |
| 4 | activity_details |  Activity details(optional) | Text(max 1000) |


The sample java implementation repository is on the https://github.com/e-permit/e-permit-java

