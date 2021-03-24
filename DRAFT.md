### Claims of e-permit offline credential


| Code | Field | Description | Required | Format | Sample Value | 
| ---- | ------| ----------- | -------- | ------ | ------------ | 
| 1 | py | Year of the permit | &#9745; | Year | 2020 |
| 3 | pid | Serial Number of the permit | &#9745; | Number | 1 |
| 4 | iss | This permit issued by |  &#9745; | Country code | ua |
| 6 | cn | Name of the company | | Text(max 100) | Sample Org. |
| 8 | sub | Plate number(s) | &#9745; | Text | 06BB2020 |
| 9 | pt | Type of the permit | &#9745; | Enum[1,2,3] | "biliteral", "transit", "3rdcountry" |
| 11 | aud | This permit issued for | &#9745; | Country code | tr |

### Claims of e-permit credential

| Code | Field | Description | Required | Format | Sample Value | 
| ---- | ------| ----------- | -------- | ------ | ------------ | 
| 1 | permit_year | Year of the permit | &#9745; | Year | 2020 |
| 2 | expire_at |  Permit valid until | &#9745; | Unix Epoch Time | 1311281970 |
| 3 | permit_id | Serial Number of the permit | &#9745; | Number | 1 |
| 4 | issuer | This permit issued by |  &#9745; | Country code | ua |
| 5 | issued_at | This permit prepared on | &#9745; | Unix Epoch Time | 1311281970 |
| 6 | company_name | Name of the company | | Text(max 100) | Sample Org. |
| 7 | company_id | National ID of the company | &#9745; | Text | A101.. | 
| 8 | plate_number | Plate number(s) | &#9745; | Text | 06BB2020 |
| 9 | permit_type | Type of the permit | &#9745; | Enum[1,2,3] | "biliteral", "transit", "3rdcountry" |
| 10 | restrictions | Restrictions | | Text(max 100) | Sample res. |
| 11 | issued_for | This permit issued for | &#9745; | Country code | tr |
| 11 | claims | Other claims | &#9745; | Key Value | {} |
