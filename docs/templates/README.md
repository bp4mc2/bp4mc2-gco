# Gebruik templates

Het template [Waardelijst-begrippen-template.xlsx](Waardelijst-begrippen-template.xlsx) bevat een excel waarmee de transformatie uitgevoerd kan worden naar een SKOS begrippenkader.

Na invullen van het template kan deze getransformeerd worden naar [CSVW](https://www.w3.org/TR/tabular-data-model/): tabular data on the web. Dit betekent dat de excel is omgezet naar een canoniek formaat, op basis waarvan de vertaling naar SKOS kan plaatsvinden.

Hiervoor kan gebruik gemaakt worden van de Excel-naar-RDF transformator, te vinden op: [linkeddata.ordina.nl/excel2rdf](http://linkeddata.ordina.nl/excel2rdf).

Het resultaat kan vervolgens als RDF bestand gedownload worden (via [deze link](http://linkeddata.ordina.nl/excel2rdf/query/download)), om vervolgens om te zetten naar SKOS met behulp van een SPARQL statement Hiervoor is het noodzakelijk om het RDF bestand in te laden in een triplestore, elke willekeurige triplestore is hiervoor bruikbaar. Let er wel op dat in het SPARQL statement expliciet een named graph URI wordt gezet (namelijk: `urn:dataset:csvw-data`). Dit moet ook de URI zijn van de named graph waar je het RDF bestand in hebt geplaatst.

Omdat we specifieke URI's willen munten (UpperCamelCase), is het eerst noodzakelijk om aan de CSVW output de gemunte URI toe te voegen, feitelijk de "primary key". Dit kan met behulp van het script [CSVWAddKey.sparql](CSVWAddKey.sparql). LET OP: Dit script voegt waarden toe aan de bestaande CSVW. Het is dan ook noodzakelijk dat je schrijfrechten hebt op het SPARQL enpoint. Bij enkele triplestores is het ook noodzakelijk om een ander endpoint te gebruiken (bijvoorbeeld `/update` in plaats van `/query`).

Hierna kan de omgezette data aangemaakt worden. De SPARQL query is hier te vinden: [CSVW2SKOS.sparql](CSVW2SKOS.sparql).

Om vervolgens dit resultaat weer als "platte" JSON te gebruiken, is het noodzakelijk om deze JSON te "framen" op de juiste manier voor de betreffende API. Hiervoor kan gebruik gemaakt worden van het volgende frame: [frame.json](frame.json). Om de JSON-LD data om te zetten kan hiervoor gebruik worden gemaakt van de [JSON-LD playground](https://json-ld.org/playground/). Kies bij de output voor het 4e tabblad "Framed". Zet aan de linkerkant je originele JSON-LD neer, en aan de rechterkant het [frame.json](frame.json). Het uiteindelijke JSON bestand is vervolgens in de output te zien.
