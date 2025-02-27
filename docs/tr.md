# E-PERMIT

## Giriş
E-Permit, ülkeler arası taşımacılık izinlerinin dijital olarak yönetilmesini sağlayan ve güvenli veri alışverişini temin eden elektronik bir izin sistemidir. Her ülke, birbirleriyle standartlaştırılmış ve güvenli bir şekilde izin bilgisi alışverişinde bulunmak amacıyla RESTful API’ler üzerinden iletişim kurar. Bu doküman; sistemin kurulumu, API endpoint’lerinin kullanımı, güvenlik, doğrulama, veri bütünlüğü ve hata yönetimi gibi kritik konuları detaylı bir şekilde ele almaktadır.

## Mimari:

E-Permit projesi, iki ana API bileşeni etrafında yapılandırılmıştır: Public API ve Internal API. Bu ayrımın temel nedeni, güvenlik ve işlevsellik açısından farklı gereksinimleri karşılamaktır.

- **Public API:**  
  Dış dünyaya açılmış olan bu API, partner ülkelerle veri alışverişi için kullanılır. Diğer ülkeler, Public API üzerinden kendi public key’lerini sunabilir, izinleri veya kullanım bilgilerini gönderebilir. Bu API, dijital imzalar ve diğer doğrulama mekanizmaları sayesinde gelen mesajların güvenilirliğini sağlar. Örneğin Türkiye için public api https://disapi.uab.gov.tr/apigateway/e-permit olarak sunulmuştur. Bu adreste Türkiye'yi temsil eden public key listesi sunulmaktadır.

- **Internal API:**  
  Kurum içindeki diğer sistemlerle entegrasyonu sağlayan bu API, yalnızca yerel sistemlerin erişimine açıktır. Internal API, kota tanımlama, izin düzenleme, iptal işlemleri gibi idari ve otomatik süreçleri yönetir.

- **Senkran Veritabanları ve Veri Alışverişi:**  
  Her iki API’de gönderilen her mesaj, önce kendi veritabanına işlenir; ardından aynı mesaj, karşı tarafın sunduğu servisler üzerinden partner ülkenin veritabanına da aktarılır. Bu mekanizma sayesinde, iki ülke arasında veri senkronizasyonu sağlanır. Ayrıca, mesajların tutarlılığı `previous_event_id` referans bilgisi kullanılarak izlenir; böylece, mesaj sıralaması ve veri bütünlüğü garanti altına alınır.

Bu mimari yaklaşım, hem yerel sistemlerin verimli çalışmasını hem de ülkeler arası veri alışverişinin kesintisiz, güvenli ve senkronize bir şekilde gerçekleşmesini sağlar.


## Kurulum ve Yapılandırma

### Open-Source Kaynak Kodu ve Docker İmajları
E-Permit sisteminin kaynak kodu GitHub üzerinde açık kaynak olarak sunulmaktadır. Gerekli durumlarda:
- **Kaynak Kodu:** [e-permit-java GitHub Repository](https://github.com/e-permit/e-permit-java) – Projeyi derlemek için JDK (21 veya daha üstü) ve Maven (3.9.9 veya daha üstü) gerekmektedir. Proje dizininde `mvn clean package` komutunu çalıştırarak JAR dosyaları üretilebilir.
- **Docker İmajları:** 
  - Public API için: `ghcr.io/e-permit/publicapi:latest`
  - Internal API için: `ghcr.io/e-permit/internalapi:latest`
  
Hazır Docker imajları, Docker Compose gibi araçlarla birlikte kullanılarak servislerin hızlıca kurulmasını sağlar.

e-permit uygulamasını kullanacak her ülke öncelikli olarak kendi  özel anahtarını (private key) üretip punlic api üzerinden de sertifikasını (public key) yayımlamalıdır.
Epermit sistemi kurulum aşamasında ilk keyi otomatik olarak oluşturmakta ve veri tabanında saklamaktadır.

## API DETAYLARI

### Public API

Public API endpointleri  karşı ülkenin internal apisi tarafından kullanılmak için hizmet verir. Bu endpointler:

*`/`* 

Her ülke kendi kimliğini temsil eden public key listesini sunar.

*`/verify/{QR Code}`*
QR kod ile düzenlenen elektronik geçiş belgesinin doğrulanmasını sağlar.


Olaylar:

*`/events/key-created`*

Üretilen public keyi karşı ülkeye sunma bu metotla yapılır.

*`/events/key-revoked`*

İptal edilen public keyi karşı ülkeye sunma bu metotla yapılır.

*`/events/quota-created`*

Karşı ülkeye permit düzenlemesi için kota tanımlama işlemi bu metod ile yapılır.

Örnek olarak Türkiye Özbekistana bir kota tanımı yaptığında Türkiye Özbekistan public apisinin  kota bilgilerini bu metoda gönderir.

*`/events/permit-created`*

Düzenlenen permiti karşı ülkeye gönderme işlemini yapar.

Örnek olarak Türkiye Özbekistanın tanımladığı kotadan kendi ülkesinin taşıtı için bir izin düzenlediğinde Özbekistan public apisinin bu metoduna  permit bilgilerini gönderir.

*`/events/permit-revoked`*

İptal edilen permiti karşı ülkeye gönderme işlemini yapar.

Örnek olarak Türkiye Özbekistanın tanımladığı kotadan kendi ülkesinin taşıtı için bir izin düzenlediğinde permit used bilgisini göndermeden ilgili iznin iptalini yapıp yeni izin düzenleyebilir. İptali yapılan permit bilgisi Özbekistan public apisinin bu metoduna  gönderilir.

*`/events/permit-used`*

Düzenlenen permitin karşı ülkeye kontrol noktalarından(sınır kapılarından) kullanımı bilgisinin (Giriş ve Çıkış) gönderimini yapar. 

Örnek olarak Türkiye Özbekistanın tanımladığı kotadan kendi ülkesinin taşıtı için bir izin düzenledikten sonra ilgili taşıt Özbekistan sınır kapılarından Giriş ve Çıkış yaptıktan sonra Özbekistan tarafından ilgili permit used bilgisi Türkiye Public apisindeki bu metoda iletilir.


### Internal API

Taraf ülkelerin kendi sistemleri tarafından kullanılması için oluşturulan api.

Kota ve permit işlemlerinin karşı ülkenin public apisi ile entegre biçimde yönetilebileceği internal api uygulamasıda open source olarak yayımlanmıştır. 
Docker image adresi: ghcr.io/e-permit/internalapi:latest

#### Örnek ortam değişkeleri:
```env
SPRING_PROFILES_ACTIVE=dev
HIBERNATE_DDL_AUTO=none
FLYWAY_ENABLED=true
FLYWAY_SCHEMAS=public
SPRING_DATASOURCE_URL=<db url>
SPRING_DATASOURCE_USERNAME=<db user>
SPRING_DATASOURCE_PASSWORD=<db pwd>
SPRING_DATASOURCE_DRIVER=<db driver>
SPRING_DATASOURCE_DIALECT=<dialect>
EPERMIT_ISSUER_CODE=<Country code>
EPERMIT_ISSUER_NAME=<Country name>
EPERMIT_ADMIN_PASSWORD=<admin pwd>
EPERMIT_KEY_PASSWORD=<admin pwd for encrypting key>
EPERMIT_LOG_BASEPATH=<log base path e.g /var/log/epermit> 
```


#### İşlevler:

El sıkışma (ülke tanımı):
Ülkelerin dijital kimliklerini birbirlerine tanıtmak için ilk aşamada el sıkışma(key tanımlama) işlemi yapılır.
Türkiye olarak Özbekistan ikili anlaşmasına istinaden Özbekistan epermit sistemini tanıtmak için; 
`/authorities`  adresine post isteği ile aşağıdaki yapıda veri gönderilir.

```json
{
  "api_uri": http://uz-public-api 
}
```

api_uri ikili olarak anlaşma yapılan ülkenin sunduğu public api servis adresi. Public api adresinde bulunan detay bilgileri ile epermit sistemine ülke tanımlaması yapılır.
Özbekistan tarafıda Türkiyeyi kendi epermit sistemine tanıtmak için aynı işlemleri yapmalıdır.
Kota Tanımı:
Anlaşmalı olarak tanımlanan karşı ülkeye geçiş belgesi dağıtımı için kota tahsis işlemi yapılmasıdır.
Örneğin Türkiye tarafı Özbek taşımacılara tahsis edilmek üzere Özbekistana geçiş belgesi kotası tahsis eder.  Belge tipleri  aşağıdaki gibidir;
`BILATERAL`,`TRANSIT`,`THIRDCOUNTRY`, `BILATERAL_FEE`,`TRANSIT_FEE`,`THIRDCOUNTRY_FEE`

`/authority_quotas` adresine aşağıdaki gibi post isteği  yapılır.
```json
{
    "authority_code": "UZ",
    "permit_type": "BILATERAL",
    "permit_year": 2024,
    "start_number": 1,
    "end_number": 250
}
```
post isteği karşı internal api üzerinden karşı ülkenin public api    
`/events/quota-created`
iletilir. Karşı taraf servise anında iletilemediği durumda kuyruğa alınır arka planda otomatik gönderimi sağlanmaktadır.
Geçiş Belgesi Düzenleme:
Anlaşmalı ülkenin tanımladığı kotadan kendi taşımacıları için elektronik geçiş belgesi düzenleme işlemi.
Örneğin Türkiye Özbekistan tarafının tanımladığı kota ile Türk taşımacılar için aşağıdaki örnekte olduğu gibi elektronik geçiş belgesi tanımı yapabilir.
`/permits` aşağıdaki gibi post isteği gönderilir.
```json
{
    "issued_for": "UZ",
    "permit_year": 2024,
    "permit_type": "BILATERAL",
    "company_name": "TECT",
    "company_id": "123",
    "plate_number": "TECT"
}
```
post isteği karşı internal api üzerinden karşı ülkenin public api    
/events/permit-created 
iletilir. Karşı taraf servise anında iletilemediği durumda kuyruğa alınır arka planda otomatik gönderimi sağlanmaktadır.
Geçiş Belgesi İptal İşlemi:
Düzenlenen elektronik geçiş belgesi eğer düzenleyen ülke tarafından iptal edilmek istenirse ilgili belgenin giriş/çıkış (used) bilgisi karşı ülkeden iletilmemişse bu metod üzerinden iptali yapılıp karşı ülkeye bildirimi yapılabilir. 

Örneğin Türkiye kullanıldı bilgisi iletilmemiş bir geçiş belgesine aşağıdaki örnekte olduğu gibi iptal bildirimi yapabilir.
`/permits/{permit-id}` aşağıdaki gibi delete isteği gönderilir.
Örneğin : `/permits/TR-UZ-2022-1-2`
  
delete isteği internal api üzerinden karşı ülkenin public api    
`/events/permit-revoked` 
iletilir. Karşı taraf servise anında iletilemediği durumda kuyruğa alınır arka planda otomatik gönderimi sağlanmaktadır.

Geçiş Belgesi Kullanım Bilgisi (Giriş-Çıkış) İşlemleri:
Taşımacı, hedef ülkeye elektronik belge ile  giriş-çıkış yaptığında geçiş belgesinin giriş-çıkış bilgisi taşımacı ülke epermit sistemine iletilmelidir.

*`/permits/{permit_id}/activities`*

Örnek: `permits/TR-UZ-2022-1-1/activities`
Örnek giriş bildirimi isteği:
```json
{
    "activity_type": "ENTRANCE",
    "activity_timestamp": 1656406166,
    "activity_details": "Giriş kapısı bilgisi"
}
```
Örnek çıkış bildirimi isteği:
```json
{
    "activity_type": "EXIT",
    "activity_timestamp": 1656406166,
    "activity_details": "Çıkış kapısı bilgisi"
}
```
Post  isteği internal api üzerinden karşı ülkenin public api    
`/events/permit-used` 
iletilir. Karşı taraf servise anında iletilemediği durumda kuyruğa alınır arka planda otomatik gönderimi sağlanmaktadır.
HATA KODLARI VE AÇIKLAMALARI:
```
Response Yapısı:
•	401 for invalid jws
•	400 for bad request
•	200 for succeed
•	422
```
```json
{
  "error_code": " aşağıdaki hata kodları ",
  "error_message": " hata mesajı "
}
```
```
Hata Kodları:
    AUTHORITY_ALREADY_EXISTS: Tanımlanmak istenen ülke zaten mevcut:
    AUTHORITY_NOT_FOUND: Karşı ülke tanımlı olmadan işlem yapılması durumunda.
    EVENT_ALREADY_EXISTS: Gönderilen event tekrr gönderilmeye çalışılırsa.
    PREVIOUS_EVENT_NOTFOUND: Olay zincirinin iki ülke arasında senkronizasyonun bozulduğu durumda.
    GENESIS_EVENT_ALREADY_EXISTS: ilk olay tekrar oluşturulması durumunda.
    KEYID_ALREADY_EXISTS: tanımlanmak istenen keyid zaten mevcutsa.
    KEY_NOTFOUND: imzalı olayın doğrulaması sırasında key bulunamadı ise.
    INSUFFICIENT_KEY: key silme durumunda hiç key tanımı kalmadıysa.
    INSUFFICIENT_PERMIT_QUOTA: kota yetersizken geçiş belgesi oluşturulması durumunda.
    INVALID_QUOTA_INTERVAL: kota tanımında kota seri numarası tanımlanan aralığın dışında ise.
    PERMITID_ALREADY_EXISTS: aynı seri numarası ile tanımlanmak istenen geçiş belgesi zaten mevcutsa.
    PERMIT_NOTFOUND: işlem yapılan geçiş belgesi bulunamadı ise.
    PERMIT_USED: iptal edilmek istenen geçiş belgesi kullanılmışsa.
    INVALID_PERMITID: permit numarası hatalı ise
```
