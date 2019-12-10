# Gebruik templates

Het template [Waardelijst-begrippen-template.xlsx](Waardelijst-begrippen-template.xlsx) bevat een excel waarmee de transformatie uitgevoerd kan worden naar een SKOS begrippenkader.

Na invullen van het template kan deze getransformeerd worden naar [CSVW](https://www.w3.org/TR/tabular-data-model/): tabular data on the web. Dit betekent dat de excel is omgezet naar een canoniek formaat, op basis waarvan de vertaling naar SKOS kan plaatsvinden.

Hiervoor kan gebruik gemaakt worden van de Excel-naar-RDF transformator, te vinden op: [linkeddata.ordina.nl/excel2rdf](http://linkeddata.ordina.nl/excel2rdf).

Het resultaat kan vervolgens als RDF bestand gedownload worden (via [deze link](http://linkeddata.ordina.nl/excel2rdf/query/download)), om vervolgens om te zetten naar SKOS met behulp van een SPARQL statement Hiervoor is het noodzakelijk om het RDF bestand in te laden in een triplestore, elke willekeurige triplestore is hiervoor bruikbaar. Let er wel op dat in het SPARQL statement expliciet een named graph URI wordt gezet (namelijk: `urn:dataset:csvw-data`). Dit moet ook de URI zijn van de named graph waar je het RDF bestand in hebt geplaatst.

De SPARQL query is hier te vinden: [CSVW2SKOS.sparql](CSVW2SKOS.sparql)
