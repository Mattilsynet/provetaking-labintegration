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
        Note over LAB,MT: Can be called several times,<br/> especially to catch updates of requisition.
        LAB->>MT: GET /requisitions
        MT->>LAB: All requisitions for lab
    else Retrieve a specific requisiton.
        LAB->>MT: GET /requisitions/1234
        MT->>LAB: A single requisition.
    end
    Note over LAB,MT: Called when sample bag has physically arrived at lab.
    LAB->>MT: POST /requisitions/1234/sample-received   
    MT->>LAB: 200 OK

    Note over LAB,MT: When the lab starts the analysis,<br/> '/analysis-started' is called to signal<br/> to MT that it is no longer possible<br/> to update the requisition.
    LAB->>MT: POST /requisitions/1234/analysis-started
    MT->>LAB: 200 OK

    Note over LAB,MT: Result of analysis,<br/> can be calle multiple times if updates are needed.
    LAB->>MT: POST /requisitions/1234/results
    MT->>LAB: 201 Created
```


## Examples

```shell
curl -X 'GET' \
  'http://sample.sample-dev.mattilsynet.io/requisitions' \
  -H 'accept: application/vnd.mattilsynet.proveta.lab+json' \
  -H 'Authorization: Bearer <TOKEN_FROM_MASKINPORTEN>'
```

