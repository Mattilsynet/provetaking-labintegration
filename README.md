# Labintegration
For exchanging data related to integration with laboratories.
Swagger documentation can be found [here](https://sample.sample-dev.mattilsynet.io/swagger-ui/index.html?urls.primaryName=Endpoints+for+lab+integration)


## Authentication
[Maskinporten](https://docs.digdir.no/docs/Maskinporten/maskinporten_summary) will be used for authentication
between Norwegian Food Safety Authority (NFSA) and the laboratory. Maskinporten is based on the oauth2 standard.

```mermaid
sequenceDiagram
    autonumber
    participant LAB as Laboratory
    participant TTP as Maskinporten
    participant MT as NFSA
    LAB-->>TTP: Acquire token
    LAB-->>MT: Use token in query.
```

This integration requires a norwegian registered business id (organisasjonsnummer).
For foreign companies that would require registering a new norwegian business, or registering a [Norwegian-registered foreign company (NUF)](https://info.altinn.no/en/start-and-run-business/planning-starting/Choosing-Legal-Structure/norwegian-branch-of-a-foreign-company-nuf/).

After acquiring the id, an account can be created in Maskinporten. With the account set up, a public/private-key pair must be created, and the public key associated with the account.

Mattilsynet must then be given the business id, and can then assign access to our APIs.

Then following the procedure detailed [here](https://docs.digdir.no/docs/Maskinporten/maskinporten_guide_apikonsument)
(Only in norwegian, but could be translated by Chome), the client must then build a
[JWT](https://docs.digdir.no/docs/Maskinporten/maskinporten_protocol_jwtgrant) used to request an access token, asking
for the correct scope.
The JWT is the signed with the private key from the previously created key pair, and exchanged for an access token using the
"urn:ietf:params:oauth:grant-type:jwt-bearer" grant type, and the "private_key_jwt" authentication method defined by
the oauth standard. The access token can the be used for all requests towards our REST APIs, by including it in the
"Authorization" header as a Bearer token (eg. "Authorization: Bearer <token>") as long as it is valid. The validity is
defined as an expiry date given by the "exp" property in the access token.

## Normal operation

```mermaid
sequenceDiagram
    autonumber
    participant LAB as Laboratory
    participant MT as NFSA
    participant GCS as Google Cloud Storage

    Note over LAB, MT: Retrieve all requisitions for lab.<br/>Can be called multiple times to capture updates.
    LAB ->> MT: GET /requisitions
    MT -->> LAB: 200 OK (List of requisitions)

    Note over LAB, MT: Retrieve a specific requisition.
    LAB ->> MT: GET /requisitions/1234
    MT -->> LAB: 200 OK (Requisition details)

    Note over LAB, MT: Notify MT that sample bag has physically arrived at lab.
    LAB ->> MT: POST /requisitions/1234/sample-received
    MT -->> LAB: 200 OK

    Note over LAB, MT: Signal that analysis has started.<br/>Requisition updates no longer possible after this.
    LAB ->> MT: POST /requisitions/1234/analysis-started
    MT -->> LAB: 200 OK

    Note over LAB, MT: Submit analysis results.<br/>Can be called multiple times for updates.
    LAB ->> MT: POST /requisitions/1234/results (filename, mediatype)
    MT -->> LAB: 201 Created (Result ID: 4321)

    alt Optional: Add attachments to results
        Note over LAB, MT: Optional step for attaching files to results.
        LAB ->> MT: POST /requisitions/1234/results/4321/attachments
        MT -->> LAB: 200 OK (Google Cloud Storage upload URL)

        Note over LAB, GCS: Upload file data to Google Cloud Storage.
        LAB ->> GCS: POST (file data) to generated URL
        GCS -->> LAB: 200 OK (File upload confirmation)
    end

    Note over LAB, MT: Signal completion of analysis.<br/>No further actions allowed on requisition.
    LAB ->> MT: POST /requisitions/1234/analysis-completed
    MT -->> LAB: 200 OK

```
<details>
<summary>Mermaid config</summary>
Code below is only for configuring the mermaid diagrams above.
<script>
  mermaid.initialize({ sequence: { showSequenceNumbers: true } });
</script>
</details>
