# Electronic Permit System

## Legacy Permit System

![w:1200](img/e-permit-old-flow.png)

## New e-Permit System

![w:1200](img/e-permit-new-flow.png)

## Implementation

### Fields
| Field Code | Description | Required | Field Format | Value | 
| ----------- | ----------- | -------- | ------- | ------- | 
| sub | Vehicle registration plate | &#9745; |  [*identifier*] | 06BB2020 |
| iss | Issuing Authority |  &#9745; | Country code | ua |
| aud | Verifier Authority | &#9745; | Country code | tr |
| exp |  Date of expiry | &#9745; | Unix Epoch Time | 1311281970 |
| iat | Date of issuance | &#9745; | Unix Epoch Time | 1311281970 |
| cid | Credential Identifier | &#9745; | Sequential number | 1 |
| cy | Year of issuance | &#9745; | year | 2020 |
| ct | Credential Type | &#9745; | enum[1,2,3] | "biliteral", "transit", "3rd" |
| oid | Organization Identifier | &#9745; | Unique organization identifier | A101.. | 
| on | Organization Name | | Free Text(max 100) | Sample Org. |
| res | Restrictions | | Free Text(max 100) | Sample res. |


### QR Code with JWS(ES256) Content

![w:1200](img/e-permit-cred-format.png)

### Sample Authority Config 

```json
{
    "id": "ua",
    "title": "Ukraine",
    "titles": {
      "iss": "Issuer Authority",
      "aud": "Verifier Authority",
      "sub": "Subject",
      "iat": "Issuance Date",
      "exp": "Expiration Date",
      "ct": "Credential Type",
      "cy": "Credential Year",
      "cid": "Credential Identifier",
      "oid": "Organization Identifier",
      "on": "Organization Name",
      "res": "Restrictions",
      "invalid_signature_message": "Signature is not valid, check it out!",
      "invalid_version_message": "Invalid version",
      "invalid_cred_message": "Invalid credential payload",
      "invalid_aud_message": "Invalid verifier",
      "invalid_exp_message": "Credential expired",
      "iss_notfound_message": "Issuer not found",
      "jwk_notfound_message": "Issuer key not found",
      "revoked_cred_message": "Revoked credential",
      "valid_signature_message": "Signature is valid",
      "ct_1": "Biliteral",
      "ct_2": "Transit",
      "ct_3": "3rd Country"
    },
    "authorities": [
      {
        "id": "ua",
        "title": "Ukraine",
        "keys": [
          {
            "kty": "EC",
            "use": "sig",
            "kid": "1",
            "crv": "P-256",
            "x": "oPmyssan_NrAlsAEuFig2FdwJmWtnfsso2rfN3wclC8",
            "y": "Sw5kgoB7fgQq7AYn7vRzOyG-g7___cp-4BHbzBQNCEU",
            "alg": "ES256"
          }
        ]
      },
      {
        "id": "tr",
        "title": "Republic Of Turkey",
        "keys": [
          {
            "kty": "EC",
            "use": "sig",
            "kid": "1",
            "crv": "P-256",
            "x": "kKUBDGuy-smxA6omYlXBotSzPVB6qKI2jRe1x9U4_kE",
            "y": "5q8JKBbFoiNuDDibs7h5zIohNvDiG70UJKq4E4n51Kg",
            "alg": "ES256"
          }
        ]
      }
    ]
  }
```

### Rest Api

```/credentials/{id}```:
   - ```post ```: post a credential into verifier database(self contained auth) 
   - ```put ``` : change credential status by verifier officer(auth required)
   - ```get ```: get credential info and status(anonymous access)

> **id** is credential hash

## Demo

- [Issuer Demo]( https://e-permit.github.io/demo/)
- [Verifier Demo(camera is required)]( https://e-permit.github.io/verify/)


