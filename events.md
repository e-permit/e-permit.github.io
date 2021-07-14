### Events

Bir ülke kota tanımı yaptığında, permit ürettiğinde-iptal ettiğinde veya karşı ülke taşımacısı ülkeye giriş yaptığında bu bilgileri karşı ülkeye göndermelidir. 
Gönderilen bu bilgiler event olarak adlandırılır. En az 4 adet event türü bulunmaktadır. Bunlar

- QUOTA_CREATED
- PERMIT_CREATED
- PERMIT_REVOKED
- PERMIT_USED

Elektronik entegrasyon yapılan her ülke, bu olayları üretip karşıya gönderebilmeli ve karşı ülke tarafından oluşturulan olayları işleyebilmelidir. 

Her olay ortak olarak belli alanları içermelidir. Bunlar 

| No | Field | Description | Details | 
| ---- | ------| ----------- | -------- | 
| 1 | event_id |  The unique identifier of event | Recommended(UUID-GUID) |
| 2 | previous_event_id | Previous event identifier |  If there is no previous event use "0" |
| 3 | event_type | Event type | Enum[QUOTA_CREATED, PERMIT_USED, PERMIT_CREATED, PERMIT_REVOKED, KEY_CREATED(Optional), KEY_REVOKED(Optional)] | 
| 4 | event_timestamp | The UTC time of the event | Long(1625304893) |
| 5 | event_issuer | Event issuer(publisher)| Two letters country code(e.g. TR, UZ, UA) |
| 6 | event_issued_for | Event audience | Two letters country code(e.g. TR, UZ, UA) |

İki ülke sistemindeki verilerin tutarlılığı için her olay kendinden bir önceki olaya işaret etmelidir. 
Bu amaçla ```previous_event_id``` alanı kullanılmaktadır. İlk olay için bu değer ```0``` olmalıdır.

#### QUOTA_CREATED 

```POST``` ```/events/quota-created``` 

Bir ülke, karşı ülke taşımacıları için belli aralıklarda seri numarası kotası tanımlayabilmelidir. Bu işlem kota tanımlaması olarak adlandırılır.
Böylelikle karşı ülke, tanımlanan kota aralığında taşımacısı için permit üretebilecektir. Bu olayın alanları aşağıdadır

| No | Field | Description | Format | 
| ---- | ------| ----------- | -------- | 
| 1 | permit_issuer | Permit issuer for the quota | UZ |
| 2 | permit_issued_for | Permit issued_for for the quota | TR |
| 3 | permit_year |  Year of the quota | 2021 |
| 4 | permit_type | Permit type of the quota | BILITERAL-TRANSIT-THIRDCOUNTRY-BILITERAL_FEE-TRANSIT_FEE-THIRDCOUNTRY_FEE |
| 5 | start_number | Start number of the quota | 20 |
| 6 | end_number | End number of the quota | 50 |

```PermitType``` alanı hangi türde permit üretilebileceğini belirtmektedir. 6 farklı permit türü bulunmaktadır. FEE ile biten türler ücretli anlamına gelmektedir. Parantez içindeki nümerik değer türün sayısal kodunu ifade eder. Permit Id üretirken bu değer kullanılır.

- BILITERAL(1)
- TRANSIT(2)
- THIRDCOUNTRY(3)
- BILITERAL_FEE(4)
- TRANSIT_FEE(5)
- THIRDCOUNTRY_FEE(6)

#### PERMIT_CREATED

```POST``` ```/events/permit-created``` 

Bir ülke kendi taşımacısı için bir permit belgesi ürettiğinde bunu karşı ülkeye göndermelidir. Bu amaç için PERMIT_CREATED olayı kullanılır.  

| No | Field | Description | Details | 
| ---- | ------| ----------- | -------- | 
| 1 | permit_id |  Permit identifier | ```Format:``` [Issuer Country Code]-[Verifier Country Code]-[Permit Year]-[Permit Type Numeric Code]-[Permit Serial Number] e.g. TR-UZ-2021-1-1 |
| 1 | permit_issuer | Permit issuer |  UZ |
| 1 | permit_issued_for | Permit issued_for | TR |
| 2 | permit_year |  Year of the permit | 2021 |
| 3 | permit_type | Permit type | BILITERAL |
| 4 | serial_number |  Serial number of permit | 1 |
| 5 | issued_at |  This permit prepared on | 03/03/2021 |
| 6 | expire_at |  Permit valid until | 31/01/2022 |
| 7 | company_name |  Year of the quota | ABC Company |
| 8 | company_id |  Company identifier | 123 |
| 9 | plate_number |  Plate Number(s) | 06TEST1234 |
| 10 | other_claims |  Optional Data | ```{"res": "The permit is restricted..."}``` |

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

#### ```/events/quota-created``` ```POST```

A country uses this resource when it creates a permit. 



#### ```/events/key-created``` ```POST```

A country uses this resource when it creates a permit. 

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

A country uses this resource when it creates a permit. 

| No | Field | Description | Format | 
| ---- | ------| ----------- | -------- | 
| 1 | key_id |  Key identifier | Text(e.g. 2) |
| 2 | revoked_at | The UTC time of revocation | Long(123444) |
