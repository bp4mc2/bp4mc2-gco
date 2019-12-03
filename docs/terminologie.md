# Terminologie

Het model van de catalogus kent een engelstalige terminologie, conform de W3c standaarden die worden gebruikt in de catalogus. Er bestaat een Nederlandstalige terminologie die als standaard wordt gezien binnen de overheid. Daarnaast wordt in de user interface van de Catalogus bepaalde terminologie gebruikt, en ook in de API is sprake van bepaalde terminologie.

Gewenst is dat deze terminologie volledig op elkaar is afgestemd en gelijk is. Om inzichtelijk te maken waar er verschillen optreden, geeft deze pagina een overzicht.

In onderstaande tabellen komt het voor dat een kolom leeg blijft. In deze situaties bestaat er geen overeenkomstige term in het betreffende onderdeel.

## Klassen

De volgende terminologie wordt gehanteerd voor klassen:

|Term (en) in de standaard|Term (nl) in het model|Term in de userinterface|Term in de API|Reden verschil|
|-------------------------|----------------------|------------------------|--------------|--------------|
| [http://www.w3.org/2000/01/rdf-schema#Resource](Resource) | Object | | | Abstracte term, alleen in het model |
| [http://www.w3.org/2004/02/skos/core#Collection](Collection) | Collectie | | | Collecties worden nu nog niet ondersteund in de UI of API |
| [http://www.w3.org/2004/02/skos/core#Concept](Concept) | Concept | Begrip, Concept | | Op de meeste plaatsen wordt in de UI "Begrip" gebruikt, behalve in "Over de catalogus" |
| [http://www.w3.org/2004/02/skos/core#ConceptScheme](Concept scheme) | Conceptschema | Domein | |
| [http://www.w3.org/ns/dcat#Catalog](Catalog) | Catalogus | | | |
| [http://www.w3.org/ns/dcat#Dataset](Dataset) | Dataset | | | Abstracte term, alleen in het model |
| [http://www.w3.org/ns/dcat#Distribution](Distribution) | | | |
| [http://www.w3.org/ns/org#Organization](Organization) | | | |
| [http://www.w3.org/ns/prov#Entity](Entity) | Voorkomen | | |
| [http://www.w3.org/ns/prov#Person](Person) | Persoon | | |

## Typering van datasets

De volgende typering van dataset wordt gehanteerd. Deze typering staat los van de standaard en kennen Catalogus-specifieke URI's:

|Term (nl) in het model|Term in de userinterface|Term in de API|Reden verschil|
|----------------------|------------------------|--------------|--------------|
| [](Informatieproduct)|Informatieproduct| | |

## Typering van assets

De volgende typering van assets wordt gehanteerd. Deze typering staat los van de standaard en kennen Catalogus-specifieke URI's:

|Term (nl) in het model|Term in de userinterface|Term in de API|Reden verschil|
|----------------------|------------------------|--------------|--------------|
| [](Conceptschema) | | | |
| [](Waardelijst) | Waardelijst | | |

## Eigenschappen

De volgende terminologie wordt gehanteerd voor eigenschappen:

### skos:Concept (Concept)

|Term (en) in de standaard|Term (nl) in het model|Term in de userinterface|Term in de API|Reden verschil|
|-------------------------|----------------------|------------------------|--------------|--------------|
| [http://www.w3.org/2000/01/rdf-schema#label](label) | label | Label | | |
| [http://www.w3.org/1999/02/22-rdf-syntax-ns#type](type) | type | Type | | |
| [http://www.w3.org/2004/02/skos/core#prefLabel](preferred label) | voorkeursterm | Term | | |
| [http://www.w3.org/2004/02/skos/core#altLabel](alternative label) | alternatieve term | Synoniemen | | |
| [http://www.w3.org/2000/01/rdf-schema#comment](comment) | uitleg | Uitleg | | |
| [http://www.w3.org/2004/02/skos/core#definition](definition) | definitie | Definitie | | |
| [http://www.w3.org/2004/02/skos/core#scopeNote](scope note) | toelichting | Toelichting | | |
| [http://purl.org/dc/elements/1.1/source](source) | tekstbron | Tekstbron | | |
| [http://purl.org/dc/terms/source](source) | bron | Bron | | |
| [http://www.w3.org/2004/02/skos/core#inScheme](is in scheme) | behoort tot | Domein | | |
| [http://purl.org/iso25964/skos-thes#broaderGeneric](broader term (generic)) | specialisatie van | Is specialisatie van | | |
| [http://purl.org/iso25964/skos-thes#narrowerGeneric](narrower term (generic)) | generalisatie van | Is generalisatie van | | |
| [http://purl.org/iso25964/skos-thes#broaderPartitive](broader term (partitive)) | onderdeel van | Is onderdeel van | | |
| [http://purl.org/iso25964/skos-thes#narrowerPartitive](narrower term (partitive)) | bestaat uit | Bestaat uit | | |
| [http://www.w3.org/2004/02/skos/core#related](has related) | gerelateerd | Gerelateerd | | |
| [http://www.w3.org/2004/02/skos/core#semanticRelation](is in semantic relation with) | semantische relatie | heeft betrekking op | | |
| [http://www.w3.org/2004/02/skos/core#closeMatch](has close match) | vrijwel overeenkomstig | Is ongeveer hetzelfde als (extern concept) | | |
| [http://www.w3.org/2004/02/skos/core#exactMatch](has exact match) | exact overeenkomstig | is hetzelfde als (extern concept) | | |
| [http://www.w3.org/2004/02/skos/core#relatedMatch](has related match) | overeenkomstig verwant | zie ook (extern concept) | | |
| [http://www.w3.org/2004/02/skos/core#broadMatch](has broader match) | overeenkomstig generieker | is breder dan (extern concept) | | |
| [http://www.w3.org/2004/02/skos/core#narrowMatch](has narrower match) | overeenkomstig specifieker | Is enger dan (in ander schema) | | |
| [http://xmlns.com/foaf/0.1/isPrimaryTopicOf](is primary topic of) | metadata | Metadata | | |
