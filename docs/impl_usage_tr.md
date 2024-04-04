# E-PERMIT KULLANIM KLAVUZU

## Mimari:
 
e-permit mimarisi 2 farklı işlevleri içeren public-internal api olarak soyutlanmıştır. Temelde ayrımın sebebi güvenliktir. Public api dışa açık bir apidir. Internal api ise iç sistemlerin entegre edildiği apidir.
Oluşturulan her mesaj kendi veri tabanına işlendikten sonra karşı tarafın sunduğu servisler üzerinden karşı tarafın veri tabanına da işlenmektedir.  Böylece tüm veriler senkron bir şekilde iki tarafta da tutulmaktadır. Bu işlemin tutarlılığı her mesajda önceki mesaj referans bilgisi de (previous_event_id) alınarak sağlanmaktadır.
 
### Public api:
Taraf ülkelerin birbirine gönderdiği mesajları işleyen api dir.
Protokole göre her ülke e-permit standartlarında(https://e-permit.github.io/) bir public api sunmalıdır.

Public api için her ülkenin ortak kullanabileceği open source bir api hazırlanmıştır.
Public api repository adresi:
https://github.com/e-permit/e-permit-java

İstenirse public api uygulamasını içeren Docker image adresi üzerinden docker image indirilerek uygulama daha pratik bir şekilde kurulabilir.
Docker image adresi: ghcr.io/e-permit/publicapi:latest

Public api kurulumu Detaylı bilgiyi  https://github.com/e-permit/e-permit-java adresinde bulabilirsiniz.

e-permit uygulamasını kullanacak her ülke öncelikli olarak kendi  özel anahtarını (private key) üretip punlic api üzerinden de sertifikasını (public key) yayımlamalıdır.
Epermit sistemi kurulum aşamasında ilk keyi otomatik olarak oluşturmakta ve veri tabanında saklamaktadır.

Public api  işlevleri:

1.	Diğer ülkelere ülkesinin public keyini sunmak.  İletilen mesajlar ülkenin özel anahtarı (private key) ile mühürlenir. Public key ile public api üzerinde doğrulaması yapılır. 
/epermit-configuration

Örnek türkiye için public key aşağıdaki adresten sunulmaktadır.
https://disapi.uab.gov.tr/apigateway/e-permit/epermit-configuration

2.	QR kod ile düzenlenen elektronik geçiş belgesinin düzenleyen ülke public apisi ile doğrulanması.
/epermit-verify/{QR Code}

3.	Diğer ülkeler tarafından  gönderilen permit ve kota taleplerini karşılamak. Bu işlevler karşı ülkenin internal apisi tarafından kullanılmaktadır.

İşlev tanımları:
/events/key-created

Üretilen public keyi karşı ülkeye sunma bu metotla yapılır.
/events/key-revoked

İptal edilen public keyi karşı ülkeye sunma bu metotla yapılır.
/events/quota-created

Karşı ülkeye permit düzenlemesi için kota tanımlama işlemi bu metod ile yapılır.

Örnek olarak Türkiye Özbekistana bir kota tanımı yaptığında Türkiye Özbekistan public apisinin  kota bilgilerini bu metoda gönderir.

/events/permit-created

Düzenlenen permiti karşı ülkeye gönderme işlemini yapar.

Örnek olarak Türkiye Özbekistanın tanımladığı kotadan kendi ülkesinin taşıtı için bir izin düzenlediğinde Özbekistan public apisinin bu metoduna  permit bilgilerini gönderir.
/events/permit-revoked

İptal edilen permiti karşı ülkeye gönderme işlemini yapar.

Örnek olarak Türkiye Özbekistanın tanımladığı kotadan kendi ülkesinin taşıtı için bir izin düzenlediğinde permit used bilgisini göndermeden ilgili iznin iptalini yapıp yeni izin düzenleyebilir. İptali yapılan permit bilgisi Özbekistan public apisinin bu metoduna  gönderilir.

/events/permit-used

Düzenlenen permitin karşı ülkeye kontrol noktalarından(sınır kapılarından) kullanımı bilgisinin (Giriş ve Çıkış) gönderimini yapar. 

Örnek olarak Türkiye Özbekistanın tanımladığı kotadan kendi ülkesinin taşıtı için bir izin düzenledikten sonra ilgili taşıt Özbekistan sınır kapılarından Giriş ve Çıkış yaptıktan sonra Özbekistan tarafından ilgili permit used bilgisi Türkiye Public apisindeki bu metoda iletilir.

İnternal api:

Taraf ülkelerin kendi sistemleri tarafından kullanılması için oluşturulan api.

Kota ve permit işlemlerinin karşı ülkenin public apisi ile entegre biçimde yönetilebileceği internal api uygulamasıda open source olarak yayımlanmıştır. 
Docker image adresi: ghcr.io/e-permit/internalapi:latest

Örnek ortam değişkeleri:

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
# Optional(if you use docker compose you can mount host volume to container volume )
EPERMIT_LOG_BASEPATH=<log base path e.g /var/log/epermit> 


Örneğin Türkiye olarak open source public api yanında internal apiyide kullanıyoruz. Kendi iç süreçlerimizdeki diğer uygulamaları(e-devlet, unet, gebos ,gümrük servisi…vb ) direkt karşı ülkenin public apisini kullanmak yerine arada Proxy olarak internal api üzerinden e-permit sistemine entegre ederek daha sürdürülebilir bir şekilde entegrasyonlarımızı sağladık.

İnternal api işlevleri:

El sıkışma (ülke tanımı):
Ülkelerin dijital kimliklerini birbirlerine tanıtmak için ilk aşamada el sıkışma(key tanımlama) işlemi yapılır.
Türkiye olarak Özbekistan ikili anlaşmasına istinaden Özbekistan epermit sistemini tanıtmak için; 
/authorities  adresine post isteği ile aşağıdaki yapıda veri gönderilir.
{"api_uri": http://uz-public-api }: api_uri ikili olarak anlaşma yapılan ülkenin sunduğu public api servis adresi. Public api adresinde bulunan detay bilgileri ile epermit sistemine ülke tanımlaması yapılır.
Özbekistan tarafıda Türkiyeyi kendi epermit sistemine tanıtmak için aynı işlemleri yapmalıdır.
Kota Tanımı:
Anlaşmalı olarak tanımlanan karşı ülkeye geçiş belgesi dağıtımı için kota tahsis işlemi yapılmasıdır.
Örneğin Türkiye tarafı Özbek taşımacılara tahsis edilmek üzere Özbekistana geçiş belgesi kotası tahsis eder.  Belge tipleri  aşağıdaki gibidir;
“BILITERAL”,”TRANSIT”,”THIRDCOUNTRY”, “BILITERAL_FEE”,”TRANSIT_FEE”,”THIRDCOUNTRY_FEE”,
/authority_quotas     adresine aşağıdaki gibi post isteği  yapılır.
{
    "authority_code": "UZ",
    "permit_type": "BILITERAL",
    "permit_year": 2024,
    "start_number": 1,
    "end_number": 250
  }
post isteği karşı internal api üzerinden karşı ülkenin public api    
/events/quota-created 
iletilir. Karşı taraf servise anında iletilemediği durumda kuyruğa alınır arka planda otomatik gönderimi sağlanmaktadır.
Geçiş Belgesi Düzenleme:
Anlaşmalı ülkenin tanımladığı kotadan kendi taşımacıları için elektronik geçiş belgesi düzenleme işlemi.
Örneğin Türkiye Özbekistan tarafının tanımladığı kota ile Türk taşımacılar için aşağıdaki örnekte olduğu gibi elektronik geçiş belgesi tanımı yapabilir.
/permits aşağıdaki gibi post isteği gönderilir.
{
    "issued_for": "UZ",
    "permit_year": 2024,
    "permit_type": "BILITERAL",
    "company_name": "TECT",
    "company_id": "123",
    "plate_number": "TECT"
  }
post isteği karşı internal api üzerinden karşı ülkenin public api    
/events/permit-created 
iletilir. Karşı taraf servise anında iletilemediği durumda kuyruğa alınır arka planda otomatik gönderimi sağlanmaktadır.
Geçiş Belgesi İptal İşlemi:
Düzenlenen elektronik geçiş belgesi eğer düzenleyen ülke tarafından iptal edilmek istenirse ilgili belgenin giriş/çıkış (used) bilgisi karşı ülkeden iletilmemişse bu metod üzerinden iptali yapılıp karşı ülkeye bildirimi yapılarak kotadan düşmemesi sağlanabilir. Böylece iptal edilen geçiş belgesi numarası tekrar kullanılabilir ve yeni bir taşımacıya tahsis edilebilir.

Örneğin Türkiye kullanıldı bilgisi iletilmemiş bir geçiş belgesine aşağıdaki örnekte olduğu gibi iptal bildirimi yapabilir.
/permits/{permit-id} aşağıdaki gibi delete isteği gönderilir.
Örneğin : /permits/TR-UZ-2022-1-2
  
delete isteği internal api üzerinden karşı ülkenin public api    
/events/permit-revoked 
iletilir. Karşı taraf servise anında iletilemediği durumda kuyruğa alınır arka planda otomatik gönderimi sağlanmaktadır.

Geçiş Belgesi Kullanım Bilgisi (Giriş-Çıkış) İşlemleri:
Taşımacı, hedef ülkeye elektronik belge ile  giriş-çıkış yaptığında geçiş belgesinin giriş-çıkış bilgisi taşımacı ülke epermit sistemine iletilmelidir.

permits/{permit_id}/activities
Örnek: permits/TR-UZ-2022-1-1/activities
Örnek giriş bildirimi isteği:
{
    "activity_type": "ENTRANCE",
    "activity_timestamp": 1656406166,
    "activity_details": "Giriş kapısı bilgisi"
  }
Örnek çıkış bildirimi isteği:
{
    "activity_type": "EXIT",
    "activity_timestamp": 1656406166,
    "activity_details": "Çıkış kapısı bilgisi"
  }
Post  isteği internal api üzerinden karşı ülkenin public api    
/events/permit-used 
iletilir. Karşı taraf servise anında iletilemediği durumda kuyruğa alınır arka planda otomatik gönderimi sağlanmaktadır.
HATA KODLARI VE AÇIKLAMALARI:
Response Yapısı:
•	401 for invalid jws
•	400 for bad request
•	200 for succeed
•	422 
{
  "error_code": " aşağıdaki hata kodları ",
	"error_message": " hata mesajı "

}
Hata Kodları:
    AUTHORITY_ALREADY_EXISTS: Tanımlanmak istenen ülke zaten mevcut:
    AUTHORITY_NOT_FOUND: Karşı ülke tanımlı olmadan işlem yapılması durumunda.
    EVENT_ALREADY_EXISTS: Gönderilen event tekrr gönderilmeye çalışılırsa.
    PREVIOUS_EVENT_NOTFOUND: Olay zincirinin iki ülke arasında senkronizasyonun           bozulduğu durumda.
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
