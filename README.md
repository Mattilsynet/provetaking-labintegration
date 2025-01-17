# Labintegration
For exchanging data related to integration with laboratories.
Swagger documentation can be found [here](https://sample.sample-dev.mattilsynet.io/swagger-ui/index.html?urls.primaryName=Endpoints+for+lab+integration)


## Authentication
[Maskinporten](https://docs.digdir.no/docs/Maskinporten/maskinporten_overordnet) will be used for authentication between Norwegian Food Safety Authority (NFSA) and the laboratory.
Maskinporten is a trusted third party, requires Norwegian organisation number.

```mermaid
sequenceDiagram
    participant LAB as Laboratory
    participant TTP as Maskinporten
    participant MT as NFSA
    LAB-->>TTP: Acquire token
    LAB-->>MT: Use token in query.
```

## Normal operation

```mermaid
sequenceDiagram
    participant LAB as Laboratory
    participant MT as NFSA
    alt Acquire all requisition
        Note right of MT: Can be called several times, especially to catch updates of requisition.
        LAB->>MT: GET /requisitions
        MT->>LAB: All requisitions for lab
    else Retrieve a specific requisiton.
        LAB->>MT: GET /requisitions/1234
        MT->>LAB: A single requisition.
    end
    LAB->>MT: POST /requisitions/1234/sample-received
    Note right of MT: Called when sample bag has physically arrived at lab.
    MT->>LAB: 200 OK
    LAB->>MT: POST /requisitions/1234/analysis-started
    Note right of MT: When the lab starts the analysis, this is called to signal to MT that it is no longer possible to update the requisition.
    MT->>LAB: 200 OK
    LAB->>MT: POST /requisitions/1234/results
    Note right of MT: Result of analysis, can be calle multiple times if updates are needed.
    MT->>LAB: 201 Created
```


## Fetching requisitions for laboratory
NFSA will make all requisitions for the laboratory available at the '/requisitions' endpoint.
Retriving all requisitons can be beneficial for getting a forcast of inbound samples.

```mermaid
sequenceDiagram
    participant LAB as Laboratory
    participant MT as NFSA
    LAB->>MT: GET /requisitions
    activate MT
    MT-->>LAB: All requisitions
    deactivate MT
```

```shell
curl -X 'GET' \
  'http://sample.sample-dev.mattilsynet.io/requisitions' \
  -H 'accept: application/vnd.mattilsynet.proveta.lab+json' \
  -H 'Authorization: Bearer <TOKEN_FROM_MASKINPORTEN>'
```

