# E-PERMIT

## Introduction
E-Permit is an electronic permit system that enables the digital management of international transportation permits and ensures secure data exchange. Each country communicates via RESTful APIs to exchange permit information in a standardized and secure manner. This document details critical aspects of the system, including installation, usage of API endpoints, security, authentication, data integrity, and error management.

## Architecture

The E-Permit project is structured around two main API components: the Public API and the Internal API. This separation is primarily to meet different requirements in terms of security and functionality.

- **Public API:**  
  This API, exposed to the external world, is used for data exchange with partner countries. Other countries can present their public keys and send permit or usage information via the Public API. The reliability of incoming messages is ensured through digital signatures and other verification mechanisms. For example, for Turkey, the public API is provided at [https://disapi.uab.gov.tr/apigateway/e-permit](https://disapi.uab.gov.tr/apigateway/e-permit), which displays the list of public keys representing Turkey.

- **Internal API:**  
  This API is designed for integration with other systems within the institution and is accessible only by local systems. The Internal API manages administrative and automated processes such as quota definition, permit issuance, and cancellation.

- **Synchronized Databases and Data Exchange:**  
  Every message sent through both APIs is first recorded in the sender’s database; then, the same message is transmitted via services provided by the partner country to their database. This mechanism ensures data synchronization between the two countries. In addition, the consistency of messages is tracked using the `previous_event_id` reference, which guarantees proper message ordering and data integrity.

This architectural approach ensures both the efficient operation of local systems and uninterrupted, secure, and synchronized data exchange between countries.

## Installation and Configuration

### Open-Source Source Code and Docker Images
The source code of the E-Permit system is available as open source on GitHub. When needed:
- **Source Code:** [e-permit-java GitHub Repository](https://github.com/e-permit/e-permit-java) – To compile the project, JDK (version 21 or higher) and Maven (version 3.9.9 or higher) are required. You can generate the JAR files by running the `mvn clean package` command in the project directory.
- **Docker Images:** 
  - For the Public API: `ghcr.io/e-permit/publicapi:latest`
  - For the Internal API: `ghcr.io/e-permit/internalapi:latest`

Pre-built Docker images can be used in conjunction with tools such as Docker Compose to quickly deploy the services.

Each country using the e-permit application should first generate its own private key and then publish its certificate (public key) via the public API. During installation, the e-permit system automatically creates the initial key and stores it in the database.

## API DETAILS

### Public API

The Public API endpoints are provided for use by the partner country’s Internal API. These endpoints include:

**`/`**  
Each country provides a list of public keys representing its identity.

**`/verify/{QR Code}`**  
Verifies the electronic transit document issued via a QR code.

#### Events:

**`/events/key-created`**  
This method is used to present the generated public key to the partner country.

**`/events/key-revoked`**  
This method is used to present a revoked public key to the partner country.

**`/events/quota-created`**  
This method is used to define a quota for permit issuance to the partner country.  
For example, when Turkey assigns a quota to Uzbekistan, Turkey sends the quota information to this method via the Uzbekistan Public API.

**`/events/permit-created`**  
This method sends the issued permit to the partner country.  
For example, when Turkey issues a permit for its vehicle based on a quota defined by Uzbekistan, the permit information is sent to the Uzbekistan Public API using this method.

**`/events/permit-revoked`**  
This method sends the cancellation of a permit to the partner country.  
For example, if Turkey cancels a permit—without sending the "permit used" information—based on a quota defined by Uzbekistan, the cancellation details are sent to the Uzbekistan Public API through this method.

**`/events/permit-used`**  
This method transmits the usage information (Entrance and Exit) of the issued permit from the control points (border gates) of the partner country.  
For example, after Turkey issues a permit based on a quota defined by Uzbekistan, once the vehicle makes an entrance and exit at the Uzbekistan border gates, the "permit used" information is relayed from Uzbekistan to this method in Turkey’s Public API.

### Internal API

The Internal API is designed for use by each country’s own systems.  
The open source Internal API application, which enables integrated management of quota and permit operations with the partner country’s Public API, is available as well.  
Docker image address: `ghcr.io/e-permit/internalapi:latest`

### Example Environment Variables:
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

For example, as Turkey, we use both the open source Public API and the Internal API. Instead of using the partner country’s Public API directly for other internal processes (e-government, UNET, GEBOS, customs service, etc.), we integrate our systems with the e-permit system via the Internal API acting as a proxy, ensuring a more sustainable integration.

#### Internal API Functions:

**Handshake (Country Definition):**  
To introduce the digital identities of countries to each other, a handshake (key registration) process is carried out as an initial step. For example, as Turkey, to introduce the Uzbekistan e-permit system based on a bilateral agreement, a POST request is sent to the `/authorities` endpoint with the following payload:

```json
{
  "api_uri": "http://uz-public-api"
}
```

Here, `api_uri` is the public API service address provided by the partner country with which the agreement has been made. Using the detailed information available at the public API address, the e-permit system performs the country definition. Similarly, Uzbekistan must perform the same steps to introduce Turkey into its own e-permit system.

**Quota Definition:**  
This process involves allocating a quota for the distribution of transit permits to the partner country as per the agreement. For example, Turkey allocates a transit permit quota to Uzbekistan for Uzbek carriers. The permit types are as follows:
`BILATERAL`, `TRANSIT`, `THIRDCOUNTRY`, `BILATERAL_FEE`, `TRANSIT_FEE`, `THIRDCOUNTRY_FEE`

A POST request is sent to the `/authority_quotas` endpoint with the following payload:

```json
{
    "authority_code": "UZ",
    "permit_type": "BILATERAL",
    "permit_year": 2024,
    "start_number": 1,
    "end_number": 250
}
```

This POST request is transmitted via the Internal API to the partner country’s Public API at `/events/quota-created`. If the message cannot be delivered immediately, it is queued and sent automatically in the background.

**Issuing a Transit Permit:**  
This is the process of issuing an electronic transit permit for local carriers from the quota defined by the partner country. For example, as Turkey, using the quota defined by Uzbekistan for Turkish carriers, an electronic transit permit is issued as shown in the following example. A POST request is sent to the `/permits` endpoint with the following payload:

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

This POST request is transmitted via the Internal API to the partner country’s Public API at `/events/permit-created`. If the message cannot be delivered immediately, it is queued and sent automatically in the background.

**Transit Permit Cancellation:**  
If an issued electronic transit permit is to be canceled by the issuing country—and if the usage (entrance/exit) information has not been received from the partner country—the cancellation can be performed using this method and the cancellation is then reported to the partner country.

For example, as Turkey, a cancellation notification can be made for a transit permit that has not yet had its usage information transmitted. A DELETE request is sent to `/permits/{permit-id}` as follows (for example: `/permits/TR-UZ-2022-1-2`).

The DELETE request is transmitted via the Internal API to the partner country’s Public API at `/events/permit-revoked`. If the message cannot be delivered immediately, it is queued and sent automatically in the background.

**Transit Permit Usage Information (Entrance-Exit) Operations:**  
When a carrier enters or exits the destination country using the electronic document, the transit permit’s entrance/exit information must be transmitted to the carrier country’s e-permit system.

**Endpoint:** `/permits/{permit_id}/activities`

Example: `permits/TR-UZ-2022-1-1/activities`

Example entrance notification request:
```json
{
    "activity_type": "ENTRANCE",
    "activity_timestamp": 1656406166,
    "activity_details": "Entrance gate information"
}
```

Example exit notification request:
```json
{
    "activity_type": "EXIT",
    "activity_timestamp": 1656406166,
    "activity_details": "Exit gate information"
}
```

This POST request is transmitted via the Internal API to the partner country’s Public API at `/events/permit-used`. If the message cannot be delivered immediately, it is queued and sent automatically in the background.

## ERROR CODES AND DESCRIPTIONS

```
Response Structure:
•  401 for invalid JWS
•  400 for bad request
•  200 for success
•  422
```

```json
{
  "error_code": " (see error codes below) ",
  "error_message": " error message "
}
```

```
Error Codes:
    AUTHORITY_ALREADY_EXISTS: The country intended to be defined already exists.
    AUTHORITY_NOT_FOUND: Operation attempted without the partner country being defined.
    EVENT_ALREADY_EXISTS: Attempt to send an event that has already been sent.
    PREVIOUS_EVENT_NOTFOUND: Disruption in the synchronization of the event chain between the two countries.
    GENESIS_EVENT_ALREADY_EXISTS: Attempt to recreate the initial event.
    KEYID_ALREADY_EXISTS: The key ID intended to be registered already exists.
    KEY_NOTFOUND: Key not found during the verification of a signed event.
    INSUFFICIENT_KEY: No key definitions remain during a key deletion process.
    INSUFFICIENT_PERMIT_QUOTA: Attempt to create a transit permit when the quota is insufficient.
    INVALID_QUOTA_INTERVAL: The quota serial number in the quota definition is outside the defined range.
    PERMITID_ALREADY_EXISTS: A transit permit with the same serial number already exists.
    PERMIT_NOTFOUND: The transit permit being processed was not found.
    PERMIT_USED: The transit permit to be canceled has already been used.
    INVALID_PERMITID: The permit number is invalid.
```
