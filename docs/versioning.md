# History, versioning and provenance

## Base provenance model
A catalog typically makes the distinction between different versions of catalog records within the catalog. This makes it possible to track at which time a specific version of a catalog record is created or updated and which person was responsible for the particular change.

The basis for the history model is the distinction between a catalog record and the topic of that particular catalog record, as depicted in the diagram below.

![](../diagram/catalogusmodel-catalogrecord.png)

The class *dcat:CatalogRecord* is specific to the DCAT vocabulary. In the generic Catalog profile, we use the more general *prov:Entity* for any record that is the primary topic of a resource in our catalog. In the same manner, we use the class *rdfs:Resource* as the more generic class of *dcat:Resource*.

## Lifecycle management: status
Catalog records go through different stages during their lifecycle. Four separate stages are supported, from the ADMS vocabulary:

- adms:UnderDevelopment
- adms:Completed
- adms:Deprecated
- adms:Withdrawn

All new catalog records have by default the status `adms:UnderDevelopment`. This means that the particular catalog record is considered *under development* and not yet available for general public.

To publish a catalog record, the status can be changed to `adms:Completed`. This means that the particular catalog record is considered *completed* and available for general public.

To mark a catalog record for deletion after it was completed (or otherwise mark the catalog record as not to be used any more), the status can be changed to `adms:Deprecated`. This means that the particular catalog record is considered *deprecated* and, although still available for general public, should not be used any more.

A catalog record that is deleted or updated after it was completed is considered *withdrawn* and gets the status `adms:Withdrawn`. Other records retain the status `adms:UnderDevelopment`.


## System History

### Specification
System history deals with the creation and deletion of catalog records in the catalog storage system. We only consider creation and deletion. "Changing" a catalog record actually indicates the creation of a new catalog record and might result in the deletion of the old catalog record.

The creation of a new catalog record results in a `prov:generatedAtTime` statement with as a value the date and time of creation.

The deletion of an existing catalog record results in a `prov:invalidatedAtTime` statement with as a value the date and time of deletion.

An update of an existing catalog record results in the creation of a new catalog record, with a `prov:wasRevisionOf` statement, relating the new catalog record with the existing catalog record. A `prov:generatedAtTime` statement is also added to the newly created catalog record.

**ONLY** in the situation of a simple system history model, a `prov:invalidatedAtTime` statement is added to the old catalog record. The values for generatedAtTime and invalidatedAtTime should be exactly the same. In the situation that this old catlaog record was already completed, the new catalog record will also get the status `adms:Completed`. This is also only in the situation of the simple system history model. The reason is that in the simple system history model, we only store the most recent version of the catalog record. So after publishing a catalog record, all changes should also directly be published.

By deleting the catalog record, you actually don't delete the topic of this catalog record. Only the catalog record itself, the data about the topic, is deleted. This implies that you can't perform a "Create" operation on the same topic after its catalog record is deleted. Instead, you should execute an update request. In such a case, the `prov:invalidatedAtTime` retains its original value (the moment of deletion).

The simple history model only tracks the changes made to a catalog record. It doesn't actually store different versions of the same catalog record. This also means that we can't retrieve older catalog records. Only the most recent catalog record is available. We can, however, retrieve the provenance of the changed that were made.

### API for the simple history model

All examples state with the http request operation, including the request body when needed, followed by the full storage result. All statements are stored in a named graph `<http://catalog.org/data`. New statements are marked with `#NEW` to make it more clear what the actual operation does. The request body is depicted in the turtle format, the storage result is depicted in the trig format. The figure below depicts all six calls T01-T06.

![](diagrams/simple-history-model.png)


#### T01 Simple: creating a resource
Creating "Some concept", a `skos:Concept` at system time 2019-01-01T12:00:00.000

```
http POST http://catalog.org/api/v1/concepts

REQUEST BODY:
[] a skos:Concept;
  rdfs:label "Some concept"@en;
.

STORAGE RESULT:
<http://catalog.org/data> {
  <http://catalog.org/id/concept/some-concept> a skos:Concept; #NEW
    rdfs:label "Some concept"@en;                              #NEW
  .
  <http://catalog.org/doc/2019-01-01T12:00:00.000/some-concept> a prov:Entity; #NEW
    prov:generatedAtTime "2019-01-01T12:00:00.000"^^xsd:dateTime;              #NEW
    adms:status admsstatus:UnderDevelopment;                                   #NEW
    prov:isPrimaryTopicOf <http://catalog.org/id/concept/some-concept>         #NEW
  .
}
```

#### T02 Simple: updating an existing resource
Change the name and adding a comment at system time 2019-01-02T12:00:00.000. An existing resource is considered to be changed "as a whole": all statements are send, even those that are not changed.

```
PUT http://catalog.org/id/concept/some-concept

REQUEST BODY
<http://catalog.org/id/concept/some-concept> a skos:Concept;
  rdfs:label "Some concept.."@en;
  rdfs:comment "A comment about some concept"@en;
.

STORAGE RESULT:
<http://catalog.org/data> {
  <http://catalog.org/id/concept/some-concept> a skos:Concept; #NEW
    rdfs:label "Some concept.."@en;                            #NEW
    rdfs:comment "A comment about some concept"@en;            #NEW
  .
  <http://catalog.org/doc/2019-01-02T12:00:00.000/some-concept> a prov:Entity;       #NEW
    prov:generatedAtTime "2019-01-02T12:00:00.000"^^xsd:dateTime;                    #NEW
    adms:status admsstatus:UnderDevelopment;                                         #NEW
    prov:isPrimaryTopicOf <http://catalog.org/id/concept/some-concept>;              #NEW
    prov:wasRevisionOf <http://catalog.org/doc/2019-01-01T12:00:00.000/some-concept> #NEW
  .
  <http://catalog.org/doc/2019-01-01T12:00:00.000/some-concept> a prov:Entity;
    prov:generatedAtTime "2019-01-01T12:00:00.000"^^xsd:dateTime;
    adms:status admsstatus:UnderDevelopment;
    prov:isPrimaryTopicOf <http://catalog.org/id/concept/some-concept>
    prov:invalidatedAtTime "2019-01-02T12:00:00.000"^^xsd:dateTime;          #NEW
  .
}
```

#### T03 Simple: deleting a resource
Deleting of "Some concept" at system time 2019-01-03T12:00:00.000.

```
http DELETE http://catalog.org/id/concept/some-concept

STORAGE RESULT:
<http://catalog.org/data> {
  <http://catalog.org/id/concept/some-concept> a skos:Concept;
    rdfs:label "Some concept.."@en;
    rdfs:comment "A comment about some concept"@en;
  .
  <http://catalog.org/doc/2019-01-02T12:00:00.000/some-concept> a prov:Entity;
    prov:generatedAtTime "2019-01-02T12:00:00.000"^^xsd:dateTime;
    adms:status admsstatus:UnderDevelopment;
    prov:isPrimaryTopicOf <http://catalog.org/id/concept/some-concept>;
    prov:wasRevisionOf <http://catalog.org/doc/2019-01-01T12:00:00.000/some-concept>
    prov:invalidatedAtTime "2019-01-03T12:00:00.000"^^xsd:dateTime;                  #NEW
  .
  <http://catalog.org/doc/2019-01-01T12:00:00.000/some-concept> a prov:Entity;
    prov:generatedAtTime "2019-01-01T12:00:00.000"^^xsd:dateTime;
    adms:status admsstatus:UnderDevelopment;
    prov:isPrimaryTopicOf <http://catalog.org/id/concept/some-concept>
    prov:invalidatedAtTime "2019-01-02T12:00:00.000"^^xsd:dateTime;
  .
}
```

#### T04 Simple: updating a resource after it was deleted
We want to "undelete" the catalog record for the resource at system time 2019-01-04T12:00:00.000.

```
http PUT http://catalog.org/id/concept/some-concept

REQUEST BODY:
<http://catalog.org/id/concept/some-concept> a skos:Concept;
  rdfs:label "Some concept.."@en;
  rdfs:comment "A comment about some concept"@en;
.

STORAGE RESULT:
<http://catalog.org/data> {
  <http://catalog.org/id/concept/some-concept> a skos:Concept; #NEW
    rdfs:label "Some concept.."@en;                            #NEW
    rdfs:comment "A comment about some concept"@en;            #NEW
  .
  <http://catalog.org/doc/2019-01-04T12:00:00.000/some-concept> a prov:Entity;       #NEW
    prov:generatedAtTime "2019-01-04T12:00:00.000"^^xsd:dateTime;                    #NEW
    adms:status admsstatus:UnderDevelopment;                                         #NEW
    prov:isPrimaryTopicOf <http://catalog.org/id/concept/some-concept>;              #NEW
    prov:wasRevisionOf <http://catalog.org/doc/2019-01-02T12:00:00.000/some-concept> #NEW
  .
  <http://catalog.org/doc/2019-01-02T12:00:00.000/some-concept> a prov:Entity;
    prov:generatedAtTime "2019-01-02T12:00:00.000"^^xsd:dateTime;
    adms:status admsstatus:UnderDevelopment;
    prov:isPrimaryTopicOf <http://catalog.org/id/concept/some-concept>;
    prov:wasRevisionOf <http://catalog.org/doc/2019-01-01T12:00:00.000/some-concept>
    prov:invalidatedAtTime "2019-01-03T12:00:00.000"^^xsd:dateTime;
  .
  <http://catalog.org/doc/2019-01-01T12:00:00.000/some-concept> a prov:Entity;
    prov:generatedAtTime "2019-01-01T12:00:00.000"^^xsd:dateTime;
    adms:status admsstatus:UnderDevelopment;
    prov:isPrimaryTopicOf <http://catalog.org/id/concept/some-concept>
    prov:invalidatedAtTime "2019-01-02T12:00:00.000"^^xsd:dateTime;
  .
}
```

#### T05 Simple: publishing a resource
At system time 2019-01-04T12:00:00.000 we have three catalog records about the same topic. Only the most recent catalog record should be available for publication, as the other catalog records are invalidated at some point in time.

We want to publish the most recent catalog record at system time 2019-01-05T12:00:00.000

```
http POST http://catalog.org/api/v1/publish?subject=http://catalog.org/id/concept/some-concept

STORAGE RESULT:
<http://catalog.org/data> {
  <http://catalog.org/id/concept/some-concept> a skos:Concept;
    rdfs:label "Some concept.."@en;
    rdfs:comment "A comment about some concept"@en;
  .
  <http://catalog.org/doc/2019-01-04T12:00:00.000/some-concept> a prov:Entity;
    prov:generatedAtTime "2019-01-04T12:00:00.000"^^xsd:dateTime;
    adms:status admsstatus:Completed;                                                #NEW
    prov:isPrimaryTopicOf <http://catalog.org/id/concept/some-concept>;
    prov:wasRevisionOf <http://catalog.org/doc/2019-01-02T12:00:00.000/some-concept>
  .
  <http://catalog.org/doc/2019-01-02T12:00:00.000/some-concept> a prov:Entity;
    prov:generatedAtTime "2019-01-02T12:00:00.000"^^xsd:dateTime;
    adms:status admsstatus:UnderDevelopment;
    prov:isPrimaryTopicOf <http://catalog.org/id/concept/some-concept>;
    prov:wasRevisionOf <http://catalog.org/doc/2019-01-01T12:00:00.000/some-concept>
    prov:invalidatedAtTime "2019-01-03T12:00:00.000"^^xsd:dateTime;
  .
  <http://catalog.org/doc/2019-01-01T12:00:00.000/some-concept> a prov:Entity;
    prov:generatedAtTime "2019-01-01T12:00:00.000"^^xsd:dateTime;
    adms:status admsstatus:UnderDevelopment;
    prov:isPrimaryTopicOf <http://catalog.org/id/concept/some-concept>
    prov:invalidatedAtTime "2019-01-02T12:00:00.000"^^xsd:dateTime;
  .
}
```

Mark that the publication of a catalog record **doesn't** result in the creation of a new catalog record. Only the most recent catalog record is updated. This also means that the publication moment isn't recorded. In the simple history model, a `dct:issued` statement could be added to the catalog record. Another option is adopting the transaction history model, described below.

#### T06 Simple: updating a catalog record after it is published
By updating a catalog record after it is published, the new catalog record is automatically published. This means that the most recent catalog record gets the status completed. At system time 2019-01-06T12:00:00.000 we update the most recent catalog record: we add a reference to some other concept.

```
http PUT http://catalog.org/id/concept/some-concept

REQUEST BODY:
<http://catalog.org/id/concept/some-concept> a skos:Concept;
  rdfs:label "Some concept.."@en;
  rdfs:comment "A comment about some concept"@en;
  rdfs:seeAlso <http://catalog.org/id/concept/some-other-concept>;
.

STORAGE RESULT:
<http://catalog.org/data> {
  <http://catalog.org/id/concept/some-concept> a skos:Concept;       #NEW
    rdfs:label "Some concept.."@en;                                  #NEW
    rdfs:comment "A comment about some concept"@en;                  #NEW
    rdfs:seeAlso <http://catalog.org/id/concept/some-other-concept>; #NEW
  .
  <http://catalog.org/doc/2019-01-06T12:00:00.000/some-concept> a prov:Entity;       #NEW
    prov:generatedAtTime "2019-01-06T12:00:00.000"^^xsd:dateTime;                    #NEW
    adms:status admsstatus:Completed;                                                #NEW
    prov:isPrimaryTopicOf <http://catalog.org/id/concept/some-concept>;              #NEW
    prov:wasRevisionOf <http://catalog.org/doc/2019-01-04T12:00:00.000/some-concept> #NEW
  .
  <http://catalog.org/doc/2019-01-04T12:00:00.000/some-concept> a prov:Entity;
    prov:generatedAtTime "2019-01-04T12:00:00.000"^^xsd:dateTime;
    adms:status admsstatus:Withdrawn;                                                #NEW
    prov:isPrimaryTopicOf <http://catalog.org/id/concept/some-concept>;
    prov:wasRevisionOf <http://catalog.org/doc/2019-01-02T12:00:00.000/some-concept>
    prov:invalidatedAtTime "2019-01-06T12:00:00.000"^^xsd:dateTime;                  #NEW
  .
  <http://catalog.org/doc/2019-01-02T12:00:00.000/some-concept> a prov:Entity;
    prov:generatedAtTime "2019-01-02T12:00:00.000"^^xsd:dateTime;
    adms:status admsstatus:UnderDevelopment;
    prov:isPrimaryTopicOf <http://catalog.org/id/concept/some-concept>;
    prov:wasRevisionOf <http://catalog.org/doc/2019-01-01T12:00:00.000/some-concept>
    prov:invalidatedAtTime "2019-01-03T12:00:00.000"^^xsd:dateTime;
  .
  <http://catalog.org/doc/2019-01-01T12:00:00.000/some-concept> a prov:Entity;
    prov:generatedAtTime "2019-01-01T12:00:00.000"^^xsd:dateTime;
    adms:status admsstatus:UnderDevelopment;
    prov:isPrimaryTopicOf <http://catalog.org/id/concept/some-concept>
    prov:invalidatedAtTime "2019-01-02T12:00:00.000"^^xsd:dateTime;
  .
}
```

## Transaction system history model
When older versions of a catalog record should be available for retrieval, the transaction system history model is needed. With this model, all versions of a catalog record are stored in separate named graphs. The most recent version is still available in the graph. The transaction system history model created exactly the same triples as with the simple system history model, but also creates additional transaction graphs.

### API for the transaction system history model

In the following sections, we will "replay" the six transaction steps from the system history model. In the STORAGE RESULT section, we will only depict the transaction graph results. Please note that the original named graph <http://catalog.org/data> will retrieve the same statements as in the first model.

Statements in the transaction graph are **never** updated or deleted. In the following sections, only the statements that were added tot the transaction graph are mentioned.

The figure below depicts all seven calls T01-T07.

![](diagrams/transaction-history-model.png)

#### T01 Creating a resource
Creating "Some concept", a `skos:Concept` at system time 2019-01-01T12:00:00.000

```
http POST http://catalog.org/api/v1/concepts

REQUEST BODY:
[] a skos:Concept;
  rdfs:label "Some concept"@en;
.

STORAGE RESULT:
<http://catalog.org/data/transaction/2019-01-01T12:00:00.000> {
  http://catalog.org/id/concept/some-concept a skos:Concept;
    rdfs:label "Some concept"@en;
  .
  http://catalog.org/doc/2019-01-01T12:00:00.000/some-concept a prov:Entity;
    prov:generatedAtTime "2019-01-01T12:00:00.000"^^xsd:dateTime;
    adms:status admsstatus:UnderDevelopment;
    prov:isPrimaryTopicOf <http://catalog.org/id/concept/some-concept>
  .
}
```

#### T02 Updating an existing resource
Change the name and adding a comment at system time 2019-01-02T12:00:00.000. An existing resource is considered to be changed "as a whole": all statements are send, even those that are not changed.

```
PUT http://catalog.org/id/concept/some-concept

REQUEST BODY
<http://catalog.org/id/concept/some-concept> a skos:Concept;
  rdfs:label "Some concept.."@en;
  rdfs:comment "A comment about some concept"@en;
.

STORAGE RESULT:
<http://catalog.org/data/transaction/2019-01-02T12:00:00.000> {
  <http://catalog.org/id/concept/some-concept> a skos:Concept;
    rdfs:label "Some concept.."@en;
    rdfs:comment "A comment about some concept"@en;
  .
  <http://catalog.org/doc/2019-01-02T12:00:00.000/some-concept> a prov:Entity;
    prov:generatedAtTime "2019-01-02T12:00:00.000"^^xsd:dateTime;
    adms:status admsstatus:UnderDevelopment;
    prov:isPrimaryTopicOf <http://catalog.org/id/concept/some-concept>;
    prov:wasRevisionOf <http://catalog.org/doc/2019-01-01T12:00:00.000/some-concept>
  .
  <http://catalog.org/doc/2019-01-01T12:00:00.000/some-concept>
    prov:invalidatedAtTime "2019-01-02T12:00:00.000"^^xsd:dateTime;
  .
}
```

#### T03 Deleting a resource
Deleting of "Some concept" at system time 2019-01-03T12:00:00.000.

```
http DELETE http://catalog.org/id/concept/some-concept

STORAGE RESULT:
<http://catalog.org/data/transaction/2019-01-03T12:00:00.000> {
  <http://catalog.org/doc/2019-01-02T12:00:00.000/some-concept>
    prov:invalidatedAtTime "2019-01-03T12:00:00.000"^^xsd:dateTime;
  .
}
```

#### T04 Updating a resource after it was deleted
We want to "undelete" the catalog record for the resource at system time 2019-01-04T12:00:00.000.

```
http PUT http://catalog.org/id/concept/some-concept

REQUEST BODY:
<http://catalog.org/id/concept/some-concept> a skos:Concept;
  rdfs:label "Some concept.."@en;
  rdfs:comment "A comment about some concept"@en;
.

STORAGE RESULT:
<http://catalog.org/data/transaction/2019-01-04T12:00:00.000> {
  <http://catalog.org/id/concept/some-concept> a skos:Concept;
    rdfs:label "Some concept.."@en;
    rdfs:comment "A comment about some concept"@en;
  .
  <http://catalog.org/doc/2019-01-04T12:00:00.000/some-concept> a prov:Entity;
    prov:generatedAtTime "2019-01-04T12:00:00.000"^^xsd:dateTime;
    adms:status admsstatus:UnderDevelopment;
    prov:isPrimaryTopicOf <http://catalog.org/id/concept/some-concept>;
    prov:wasRevisionOf <http://catalog.org/doc/2019-01-02T12:00:00.000/some-concept>
  .
}
```

#### T05 Publishing a resource
At system time 2019-01-04T12:00:00.000 we have three catalog records about the same topic. Only the most recent catalog record should be available for publication, as the other catalog records are invalidated at some point in time.

We want to publish the most recent catalog record at system time 2019-01-05T12:00:00.000

```
http POST http://catalog.org/api/v1/publish?subject=http://catalog.org/id/concept/some-concept

STORAGE RESULT:
<http://catalog.org/data/transaction/2019-01-05T12:00:00.000> {
  <http://catalog.org/doc/2019-01-04T12:00:00.000/some-concept>
    adms:status admsstatus:Completed;
  .
}
```

#### T06 Updating a catalog record after it is published
By updating a catalog record after it is published, the new catalog record gets the status under development. This means that the most recent catalog record is not published. However, we can still retrieve the most recently published catalog record. This catalog record is stored in the named graph <http://catalog.org/data/transaction/2019-01-04T12:00:00.000>. The published catalog record remains the same and is not invalidated or withdrawn as is the case in the simple system history model. At system time 2019-01-06T12:00:00.000 we update the most recent catalog record: we add a reference to some other concept.

The main storage graph <http://catalog.org/data> will in this case differ from the main storage graph in the simple system history model. The new graph is depicted in the example below.

```
http PUT http://catalog.org/id/concept/some-concept

REQUEST BODY:
<http://catalog.org/id/concept/some-concept> a skos:Concept;
  rdfs:label "Some concept.."@en;
  rdfs:comment "A comment about some concept"@en;
  rdfs:seeAlso <http://catalog.org/id/concept/some-other-concept>;
.

STORAGE RESULT:
<http://catalog.org/data/transaction/2019-01-06T12:00:00.000> {
  <http://catalog.org/id/concept/some-concept> a skos:Concept;
    rdfs:label "Some concept.."@en;
    rdfs:comment "A comment about some concept"@en;
    rdfs:seeAlso <http://catalog.org/id/concept/some-other-concept>;
  .
  <http://catalog.org/doc/2019-01-06T12:00:00.000/some-concept> a prov:Entity;
    prov:generatedAtTime "2019-01-06T12:00:00.000"^^xsd:dateTime;
    adms:status admsstatus:UnderDevelopment;
    prov:isPrimaryTopicOf <http://catalog.org/id/concept/some-concept>;
    prov:wasRevisionOf <http://catalog.org/doc/2019-01-04T12:00:00.000/some-concept>
  .
}
<http://catalog.org/data> {
  <http://catalog.org/id/concept/some-concept> a skos:Concept;       #NEW
    rdfs:label "Some concept.."@en;                                  #NEW
    rdfs:comment "A comment about some concept"@en;                  #NEW
    rdfs:seeAlso <http://catalog.org/id/concept/some-other-concept>; #NEW
  .
  <http://catalog.org/doc/2019-01-06T12:00:00.000/some-concept> a prov:Entity;       #NEW
    prov:generatedAtTime "2019-01-06T12:00:00.000"^^xsd:dateTime;                    #NEW
    adms:status admsstatus:UnderDevelopment;                                         #NEW
    prov:isPrimaryTopicOf <http://catalog.org/id/concept/some-concept>;              #NEW
    prov:wasRevisionOf <http://catalog.org/doc/2019-01-04T12:00:00.000/some-concept> #NEW
  .
  <http://catalog.org/doc/2019-01-04T12:00:00.000/some-concept> a prov:Entity;
    prov:generatedAtTime "2019-01-04T12:00:00.000"^^xsd:dateTime;
    adms:status admsstatus:Completed;
    prov:isPrimaryTopicOf <http://catalog.org/id/concept/some-concept>;
    prov:wasRevisionOf <http://catalog.org/doc/2019-01-02T12:00:00.000/some-concept>
  .
  <http://catalog.org/doc/2019-01-02T12:00:00.000/some-concept> a prov:Entity;
    prov:generatedAtTime "2019-01-02T12:00:00.000"^^xsd:dateTime;
    adms:status admsstatus:UnderDevelopment;
    prov:isPrimaryTopicOf <http://catalog.org/id/concept/some-concept>;
    prov:wasRevisionOf <http://catalog.org/doc/2019-01-01T12:00:00.000/some-concept>
    prov:invalidatedAtTime "2019-01-03T12:00:00.000"^^xsd:dateTime;
  .
  <http://catalog.org/doc/2019-01-01T12:00:00.000/some-concept> a prov:Entity;
    prov:generatedAtTime "2019-01-01T12:00:00.000"^^xsd:dateTime;
    adms:status admsstatus:UnderDevelopment;
    prov:isPrimaryTopicOf <http://catalog.org/id/concept/some-concept>
    prov:invalidatedAtTime "2019-01-02T12:00:00.000"^^xsd:dateTime;
  .
}
```

#### Retrieving a catalog record, using its status as a filter
In the transaction system history model, we can retrieve multiple catalog records. By default, the most recent catalog record is retrieved, irrespective of its status:

```
http GET http://catalog.org/id/concept/some-concept

RESPONSE:
<http://catalog.org/id/concept/some-concept> a skos:Concept;
  rdfs:label "Some concept.."@en;
  rdfs:comment "A comment about some concept"@en;
  rdfs:seeAlso <http://catalog.org/id/concept/some-other-concept>;
.
<http://catalog.org/doc/2019-01-06T12:00:00.000/some-concept> a prov:Entity;
  prov:generatedAtTime "2019-01-06T12:00:00.000"^^xsd:dateTime;
  adms:status admsstatus:UnderDevelopment;
  prov:isPrimaryTopicOf <http://catalog.org/id/concept/some-concept>;
  prov:wasRevisionOf <http://catalog.org/doc/2019-01-04T12:00:00.000/some-concept>
.
```

To get the most recent completed catalog record, add a status-filter:

```
http GET http://catalog.org/id/concept/some-concept?status=Completed

RESPONSE
<http://catalog.org/id/concept/some-concept> a skos:Concept;
  rdfs:label "Some concept.."@en;
  rdfs:comment "A comment about some concept"@en;
.
<http://catalog.org/doc/2019-01-04T12:00:00.000/some-concept> a prov:Entity;
  prov:generatedAtTime "2019-01-04T12:00:00.000"^^xsd:dateTime;
  adms:status admsstatus:Completed;
  prov:isPrimaryTopicOf <http://catalog.org/id/concept/some-concept>;
  prov:wasRevisionOf <http://catalog.org/doc/2019-01-02T12:00:00.000/some-concept>
.
```

#### T07 Publishing a resource that has been published before
Publishing a resource for the first time, only changes the catalog record that is published. In the simple system history model, you can't publish a resource for a second time: the catalog record remains published. In the transaction system history model you can publish a resource for a second time. After publishing the new catalog record, the old published record will get the status `adms:Withdrawn` and a `prov:invalidatedAtTime`.

The value for `prov:invalidatedAtTime` is always the system time. This means that the `prov:invalidatedAtTime` for the first published catalog record is actually after the `prov:generatedAtTime` of the second published catalog record. This indicates that in this time period, two versions of a catalog record exist.

```
http POST http://catalog.org/api/v1/publish?subject=http://catalog.org/id/concept/some-concept

STORAGE RESULT:
<http://catalog.org/data/transaction/2019-01-07T12:00:00.000> {
  <http://catalog.org/doc/2019-01-06T12:00:00.000/some-concept>
    adms:status admsstatus:Completed;
  .
  <http://catalog.org/doc/2019-01-04T12:00:00.000/some-concept>
    adms:status admsstatus:Withdrawn;
    prov:invalidatedAtTime "2019-01-07T12:00:00.000"^^xsd:dateTime;
  .
}
<http://catalog.org/data> {
  <http://catalog.org/id/concept/some-concept> a skos:Concept;
    rdfs:label "Some concept.."@en;
    rdfs:comment "A comment about some concept"@en;
    rdfs:seeAlso <http://catalog.org/id/concept/some-other-concept>;
  .
  <http://catalog.org/doc/2019-01-06T12:00:00.000/some-concept> a prov:Entity;
    prov:generatedAtTime "2019-01-06T12:00:00.000"^^xsd:dateTime;
    adms:status admsstatus:Completed;                                                #NEW
    prov:isPrimaryTopicOf <http://catalog.org/id/concept/some-concept>;
    prov:wasRevisionOf <http://catalog.org/doc/2019-01-04T12:00:00.000/some-concept>
  .
  <http://catalog.org/doc/2019-01-04T12:00:00.000/some-concept> a prov:Entity;
    prov:generatedAtTime "2019-01-04T12:00:00.000"^^xsd:dateTime;
    adms:status admsstatus:Withdrawn;                                                #NEW
    prov:isPrimaryTopicOf <http://catalog.org/id/concept/some-concept>;
    prov:wasRevisionOf <http://catalog.org/doc/2019-01-02T12:00:00.000/some-concept>
    prov:invalidatedAtTime "2019-01-07T12:00:00.000"^^xsd:dateTime;                  #NEW
  .
  <http://catalog.org/doc/2019-01-02T12:00:00.000/some-concept> a prov:Entity;
    prov:generatedAtTime "2019-01-02T12:00:00.000"^^xsd:dateTime;
    adms:status admsstatus:UnderDevelopment;
    prov:isPrimaryTopicOf <http://catalog.org/id/concept/some-concept>;
    prov:wasRevisionOf <http://catalog.org/doc/2019-01-01T12:00:00.000/some-concept>
    prov:invalidatedAtTime "2019-01-03T12:00:00.000"^^xsd:dateTime;
  .
  <http://catalog.org/doc/2019-01-01T12:00:00.000/some-concept> a prov:Entity;
    prov:generatedAtTime "2019-01-01T12:00:00.000"^^xsd:dateTime;
    adms:status admsstatus:UnderDevelopment;
    prov:isPrimaryTopicOf <http://catalog.org/id/concept/some-concept>
    prov:invalidatedAtTime "2019-01-02T12:00:00.000"^^xsd:dateTime;
  .
}
```

### Retrieving a catalog record from the past
After system time 2019-01-07T12:00:00.000, the most recent catalog record is the same as the most recent completed catalog record. Sometimes it might be necessary to retrieve a catalog record as it was retrieved at a point in the past. For example at system time 2019-01-07T12:00:00.000 we will get the following result for the retrieval of the most recent completed catalog record:

```
http GET http://catalog.org/id/concept/some-concept?status=Completed

RESPONSE:
<http://catalog.org/id/concept/some-concept> a skos:Concept;
  rdfs:label "Some concept.."@en;
  rdfs:comment "A comment about some concept"@en;
  rdfs:seeAlso <http://catalog.org/id/concept/some-other-concept>;
.
<http://catalog.org/doc/2019-01-06T12:00:00.000/some-concept> a prov:Entity;
  prov:generatedAtTime "2019-01-06T12:00:00.000"^^xsd:dateTime;
  adms:status admsstatus:Completed;
  prov:isPrimaryTopicOf <http://catalog.org/id/concept/some-concept>;
  prov:wasRevisionOf <http://catalog.org/doc/2019-01-04T12:00:00.000/some-concept>
.
```

...but until system time 2019-01-07T12:00:00.000 we would get another result. To get the "old" completed catalog record, we need a filter:

```
http GET http://catalog.org/id/concept/some-concept?status=Completed&retrievedAt=2019-01-07T10:00:00.000

<http://catalog.org/id/concept/some-concept> a skos:Concept;
  rdfs:label "Some concept.."@en;
  rdfs:comment "A comment about some concept"@en;
.
<http://catalog.org/doc/2019-01-04T12:00:00.000/some-concept> a prov:Entity;
  prov:generatedAtTime "2019-01-04T12:00:00.000"^^xsd:dateTime;
  adms:status admsstatus:Completed;
  prov:isPrimaryTopicOf <http://catalog.org/id/concept/some-concept>;
  prov:wasRevisionOf <http://catalog.org/doc/2019-01-02T12:00:00.000/some-concept>
.
```

## Valid History

A more complex history model can be obtained by adding temporal characteristics to the catalog record. This results in a valid & system history model. A valid & system history model implies a transactional history model. The temporal characteristics are always about the catalog record itself: at what point in time was the catalog record considered "valid".

By adding a `dct:temporal` statement to the catalog record, we can state at which point in time the catalog record was valid and at which time the catalog record wasn't valid any more. These temporal characteristics are an extension of the status of a catalog record. A catalog record can be completed, but not yet valid. Any catalog record that is made valid is also regarded as completed. It is not possible to create a catalog record that is valid but not yet published.

![](../diagram/catalogusmodel-catalogrecord-validity.png)

The introduction of valid history created to kinds of history: valid history and system history. At any point in system-time multiple catalog records could exists, each with its own temporal characteristics. Only one of them is "the most recent" in system-time. In valid-time only one catalog record should be valid at any point in valid-time.

Marking a catalog record as valid will result in a couple of actions:

1. The status of the catalog record will be set to completed, if it is not already completed;
2. Any other catalog record that is completed but not marked as valid, is withdrawn (as a result of action 1);
3. The catalog record receives a `dct:temporal` with the start date of the validity;
4. Any old catalog record that is valid in the period of this catalog record will receive an end date of validity, and may also receive a start date of validity. If the validity of an old catalog record completely overlaps the validity of the new catalog record, the validity period of the old catalog record is set to zero (the enddata is set to the startdate).

### API for a validity history model
A validity history model implies a transaction model, it also includes system history.

The figure below depicts all five calls T08-T12.

![](diagrams/valid-history-model.png)

#### T08 Mark as valid
Making the most recent catalog record valid from december first, 2018 at system time 2019-01-08T12:00:00.000. Mark that the validity of the catalog record actually is before it was initially created. This is quite common. The oposite can also occur.

```
http POST http://catalog.org/api/v1/markvalid?subject=http://catalog.org/id/concept/some-concept&start=2018-12-01

STORAGE RESULT:
<http://catalog.org/data/transaction/2019-01-08T12:00:00.000> {
  <http://catalog.org/doc/2019-01-06T12:00:00.000/some-concept>
    dct:temporal [
      dcmiperiod:start "2018-12-01"^^xsd:date;
    ];
  .
}
<http://catalog.org/data> {
  <http://catalog.org/id/concept/some-concept> a skos:Concept;
    rdfs:label "Some concept.."@en;
    rdfs:comment "A comment about some concept"@en;
    rdfs:seeAlso <http://catalog.org/id/concept/some-other-concept>;
  .
  <http://catalog.org/doc/2019-01-06T12:00:00.000/some-concept> a prov:Entity;
    prov:generatedAtTime "2019-01-06T12:00:00.000"^^xsd:dateTime;
    dct:temporal [                                                                   #NEW
      dcmiperiod:start "2018-12-01"^^xsd:date;                                       #NEW
    ];                                                                               #NEW
    adms:status admsstatus:Completed;
    prov:isPrimaryTopicOf <http://catalog.org/id/concept/some-concept>;
    prov:wasRevisionOf <http://catalog.org/doc/2019-01-04T12:00:00.000/some-concept>
  .
  <http://catalog.org/doc/2019-01-04T12:00:00.000/some-concept> a prov:Entity;
    prov:generatedAtTime "2019-01-04T12:00:00.000"^^xsd:dateTime;
    adms:status admsstatus:Withdrawn;
    prov:isPrimaryTopicOf <http://catalog.org/id/concept/some-concept>;
    prov:wasRevisionOf <http://catalog.org/doc/2019-01-02T12:00:00.000/some-concept>
    prov:invalidatedAtTime "2019-01-07T12:00:00.000"^^xsd:dateTime;
  .
  <http://catalog.org/doc/2019-01-02T12:00:00.000/some-concept> a prov:Entity;
    prov:generatedAtTime "2019-01-02T12:00:00.000"^^xsd:dateTime;
    adms:status admsstatus:UnderDevelopment;
    prov:isPrimaryTopicOf <http://catalog.org/id/concept/some-concept>;
    prov:wasRevisionOf <http://catalog.org/doc/2019-01-01T12:00:00.000/some-concept>
    prov:invalidatedAtTime "2019-01-03T12:00:00.000"^^xsd:dateTime;
  .
  <http://catalog.org/doc/2019-01-01T12:00:00.000/some-concept> a prov:Entity;
    prov:generatedAtTime "2019-01-01T12:00:00.000"^^xsd:dateTime;
    adms:status admsstatus:UnderDevelopment;
    prov:isPrimaryTopicOf <http://catalog.org/id/concept/some-concept>
    prov:invalidatedAtTime "2019-01-02T12:00:00.000"^^xsd:dateTime;
  .
}
```

#### T09 Updating a catalog record after it was marked valid
This is actually the same as updating a catalog record after it was completed. We add a dutch label to the resource at system time 2019-01-09T12:00:00.000. We also delete the `rdfs:seeAlso` statement.

```
http PUT http://catalog.org/id/concept/some-concept

REQUEST BODY:
<http://catalog.org/id/concept/some-concept> a skos:Concept;
  rdfs:label "Some concept.."@en;
  rdfs:label "Enig begrip.."@nl;
  rdfs:comment "A comment about some concept"@en;
.

STORAGE RESULT:
<http://catalog.org/data/transaction/2019-01-09T12:00:00.000> {
  <http://catalog.org/id/concept/some-concept> a skos:Concept;
    rdfs:label "Some concept.."@en;
    rdfs:label "Enig begrip.."@nl;
    rdfs:comment "A comment about some concept"@en;
  .
  <http://catalog.org/doc/2019-01-09T12:00:00.000/some-concept> a prov:Entity;
    prov:generatedAtTime "2019-01-09T12:00:00.000"^^xsd:dateTime;
    adms:status admsstatus:UnderDevelopment;
    prov:isPrimaryTopicOf <http://catalog.org/id/concept/some-concept>;
    prov:wasRevisionOf <http://catalog.org/doc/2019-01-06T12:00:00.000/some-concept>
  .
}
<http://catalog.org/data> {
  <http://catalog.org/id/concept/some-concept> a skos:Concept; #NEW
    rdfs:label "Some concept.."@en;                            #NEW
    rdfs:label "Enig begrip.."@nl;                             #NEW
    rdfs:comment "A comment about some concept"@en;            #NEW
  .
  <http://catalog.org/doc/2019-01-09T12:00:00.000/some-concept> a prov:Entity;       #NEW
    prov:generatedAtTime "2019-01-09T12:00:00.000"^^xsd:dateTime;                    #NEW
    adms:status admsstatus:UnderDevelopment;                                         #NEW
    prov:isPrimaryTopicOf <http://catalog.org/id/concept/some-concept>;              #NEW
    prov:wasRevisionOf <http://catalog.org/doc/2019-01-06T12:00:00.000/some-concept> #NEW
  .
  <http://catalog.org/doc/2019-01-06T12:00:00.000/some-concept> a prov:Entity;
    prov:generatedAtTime "2019-01-06T12:00:00.000"^^xsd:dateTime;
    dct:temporal [
      dcmiperiod:start "2018-12-01"^^xsd:date;
    ];
    adms:status admsstatus:Completed;
    prov:isPrimaryTopicOf <http://catalog.org/id/concept/some-concept>;
    prov:wasRevisionOf <http://catalog.org/doc/2019-01-04T12:00:00.000/some-concept>
  .
  <http://catalog.org/doc/2019-01-04T12:00:00.000/some-concept> a prov:Entity;
    prov:generatedAtTime "2019-01-04T12:00:00.000"^^xsd:dateTime;
    adms:status admsstatus:Withdrawn;
    prov:isPrimaryTopicOf <http://catalog.org/id/concept/some-concept>;
    prov:wasRevisionOf <http://catalog.org/doc/2019-01-02T12:00:00.000/some-concept>
    prov:invalidatedAtTime "2019-01-07T12:00:00.000"^^xsd:dateTime;
  .
  <http://catalog.org/doc/2019-01-02T12:00:00.000/some-concept> a prov:Entity;
    prov:generatedAtTime "2019-01-02T12:00:00.000"^^xsd:dateTime;
    adms:status admsstatus:UnderDevelopment;
    prov:isPrimaryTopicOf <http://catalog.org/id/concept/some-concept>;
    prov:wasRevisionOf <http://catalog.org/doc/2019-01-01T12:00:00.000/some-concept>
    prov:invalidatedAtTime "2019-01-03T12:00:00.000"^^xsd:dateTime;
  .
  <http://catalog.org/doc/2019-01-01T12:00:00.000/some-concept> a prov:Entity;
    prov:generatedAtTime "2019-01-01T12:00:00.000"^^xsd:dateTime;
    adms:status admsstatus:UnderDevelopment;
    prov:isPrimaryTopicOf <http://catalog.org/id/concept/some-concept>
    prov:invalidatedAtTime "2019-01-02T12:00:00.000"^^xsd:dateTime;
  .
}
```

#### T10 Publishing a catalog record, with a valid catalog record already available
In the case of the transaction system history model, the second publishing of a catalog record results in the withdrawal of the first published catalog record. In the case of the valid history model, this will **only** happen when such a published catalog record has not yet made valid: valid catalog records **cannot** been withdrawn, they can only made invalid by making some other catalog record valid in the same period.

At system time 2019-01-10T12:00:00.000 we publish the most recent catalog record:

```
http POST http://catalog.org/api/v1/publish?subject=http://catalog.org/id/concept/some-concept

STORAGE RESULT:
<http://catalog.org/data/transaction/2019-01-10T12:00:00.000> {
  <http://catalog.org/doc/2019-01-09T12:00:00.000/some-concept> a prov:Entity;
    adms:status admsstatus:Completed;
  .
}
<http://catalog.org/data> {
  <http://catalog.org/id/concept/some-concept> a skos:Concept;
    rdfs:label "Some concept.."@en;
    rdfs:label "Enig begrip.."@nl;
    rdfs:comment "A comment about some concept"@en;
  .
  <http://catalog.org/doc/2019-01-09T12:00:00.000/some-concept> a prov:Entity;
    prov:generatedAtTime "2019-01-09T12:00:00.000"^^xsd:dateTime;
    adms:status admsstatus:Completed;                                                #NEW
    prov:isPrimaryTopicOf <http://catalog.org/id/concept/some-concept>;
    prov:wasRevisionOf <http://catalog.org/doc/2019-01-06T12:00:00.000/some-concept>
  .
  <http://catalog.org/doc/2019-01-06T12:00:00.000/some-concept> a prov:Entity;
    prov:generatedAtTime "2019-01-06T12:00:00.000"^^xsd:dateTime;
    dct:temporal [
      dcmiperiod:start "2018-12-01"^^xsd:date;
    ];
    adms:status admsstatus:Completed;
    prov:isPrimaryTopicOf <http://catalog.org/id/concept/some-concept>;
    prov:wasRevisionOf <http://catalog.org/doc/2019-01-04T12:00:00.000/some-concept>
  .
  <http://catalog.org/doc/2019-01-04T12:00:00.000/some-concept> a prov:Entity;
    prov:generatedAtTime "2019-01-04T12:00:00.000"^^xsd:dateTime;
    adms:status admsstatus:Withdrawn;
    prov:isPrimaryTopicOf <http://catalog.org/id/concept/some-concept>;
    prov:wasRevisionOf <http://catalog.org/doc/2019-01-02T12:00:00.000/some-concept>
    prov:invalidatedAtTime "2019-01-07T12:00:00.000"^^xsd:dateTime;
  .
  <http://catalog.org/doc/2019-01-02T12:00:00.000/some-concept> a prov:Entity;
    prov:generatedAtTime "2019-01-02T12:00:00.000"^^xsd:dateTime;
    adms:status admsstatus:UnderDevelopment;
    prov:isPrimaryTopicOf <http://catalog.org/id/concept/some-concept>;
    prov:wasRevisionOf <http://catalog.org/doc/2019-01-01T12:00:00.000/some-concept>
    prov:invalidatedAtTime "2019-01-03T12:00:00.000"^^xsd:dateTime;
  .
  <http://catalog.org/doc/2019-01-01T12:00:00.000/some-concept> a prov:Entity;
    prov:generatedAtTime "2019-01-01T12:00:00.000"^^xsd:dateTime;
    adms:status admsstatus:UnderDevelopment;
    prov:isPrimaryTopicOf <http://catalog.org/id/concept/some-concept>
    prov:invalidatedAtTime "2019-01-02T12:00:00.000"^^xsd:dateTime;
  .
}
```

#### T11 Updating a catalog record after it was marked valid and another was published
To have full understanding about the complexity, we update the catalog record again: we delete the dutch label at system time 2019-01-11T12:00:00.000:

```
http PUT http://catalog.org/id/concept/some-concept

REQUEST BODY:
<http://catalog.org/id/concept/some-concept> a skos:Concept;
  rdfs:label "Some concept.."@en;
  rdfs:comment "A comment about some concept"@en;
.

STORAGE RESULT:
<http://catalog.org/data/transaction/2019-01-11T12:00:00.000> {
  <http://catalog.org/id/concept/some-concept> a skos:Concept;
    rdfs:label "Some concept.."@en;
    rdfs:comment "A comment about some concept"@en;
  .
  <http://catalog.org/doc/2019-01-11T12:00:00.000/some-concept> a prov:Entity;
    prov:generatedAtTime "2019-01-11T12:00:00.000"^^xsd:dateTime;
    adms:status admsstatus:UnderDevelopment;
    prov:isPrimaryTopicOf <http://catalog.org/id/concept/some-concept>;
    prov:wasRevisionOf <http://catalog.org/doc/2019-01-09T12:00:00.000/some-concept>
  .
}
<http://catalog.org/data> {
  <http://catalog.org/id/concept/some-concept> a skos:Concept; #NEW
    rdfs:label "Some concept.."@en;                            #NEW
    rdfs:comment "A comment about some concept"@en;            #NEW
  .
  <http://catalog.org/doc/2019-01-11T12:00:00.000/some-concept> a prov:Entity;       #NEW
    prov:generatedAtTime "2019-01-11T12:00:00.000"^^xsd:dateTime;                    #NEW
    adms:status admsstatus:UnderDevelopment;                                         #NEW
    prov:isPrimaryTopicOf <http://catalog.org/id/concept/some-concept>;              #NEW
    prov:wasRevisionOf <http://catalog.org/doc/2019-01-09T12:00:00.000/some-concept> #NEW
  .
  <http://catalog.org/doc/2019-01-09T12:00:00.000/some-concept> a prov:Entity;
    prov:generatedAtTime "2019-01-09T12:00:00.000"^^xsd:dateTime;
    adms:status admsstatus:Completed;
    prov:isPrimaryTopicOf <http://catalog.org/id/concept/some-concept>;
    prov:wasRevisionOf <http://catalog.org/doc/2019-01-06T12:00:00.000/some-concept>
  .
  <http://catalog.org/doc/2019-01-06T12:00:00.000/some-concept> a prov:Entity;
    prov:generatedAtTime "2019-01-06T12:00:00.000"^^xsd:dateTime;
    dct:temporal [
      dcmiperiod:start "2018-12-01"^^xsd:date;
    ];
    adms:status admsstatus:Completed;
    prov:isPrimaryTopicOf <http://catalog.org/id/concept/some-concept>;
    prov:wasRevisionOf <http://catalog.org/doc/2019-01-04T12:00:00.000/some-concept>
  .
  <http://catalog.org/doc/2019-01-04T12:00:00.000/some-concept> a prov:Entity;
    prov:generatedAtTime "2019-01-04T12:00:00.000"^^xsd:dateTime;
    adms:status admsstatus:Withdrawn;
    prov:isPrimaryTopicOf <http://catalog.org/id/concept/some-concept>;
    prov:wasRevisionOf <http://catalog.org/doc/2019-01-02T12:00:00.000/some-concept>
    prov:invalidatedAtTime "2019-01-07T12:00:00.000"^^xsd:dateTime;
  .
  <http://catalog.org/doc/2019-01-02T12:00:00.000/some-concept> a prov:Entity;
    prov:generatedAtTime "2019-01-02T12:00:00.000"^^xsd:dateTime;
    adms:status admsstatus:UnderDevelopment;
    prov:isPrimaryTopicOf <http://catalog.org/id/concept/some-concept>;
    prov:wasRevisionOf <http://catalog.org/doc/2019-01-01T12:00:00.000/some-concept>
    prov:invalidatedAtTime "2019-01-03T12:00:00.000"^^xsd:dateTime;
  .
  <http://catalog.org/doc/2019-01-01T12:00:00.000/some-concept> a prov:Entity;
    prov:generatedAtTime "2019-01-01T12:00:00.000"^^xsd:dateTime;
    adms:status admsstatus:UnderDevelopment;
    prov:isPrimaryTopicOf <http://catalog.org/id/concept/some-concept>
    prov:invalidatedAtTime "2019-01-02T12:00:00.000"^^xsd:dateTime;
  .
}
```

#### T12 Marking the most recent catalog record valid
At system time 2019-01-11T12:00:00.000 we have two catalog records that could be marked valid: one has status "Completed" and one has status "UnderDevelopment". As the `markedvalid` API will by default use the most recent catalog record, only one with status "UnderDevelopment" will be used. You will need a filter to mark the one with status "Completed" (`?status=Completed`). We will mark the most recent catalog record as valid, because this is the most complex situation, at system time 2019-01-12T12:00:00.000:

```
http POST http://catalog.org/api/v1/markvalid?subject=http://catalog.org/id/concept/some-concept&start=2019-01-01

<http://catalog.org/data/transaction/2019-01-12T12:00:00.000> {
}
<http://catalog.org/data> {
  <http://catalog.org/id/concept/some-concept> a skos:Concept;
    rdfs:label "Some concept.."@en;
    rdfs:comment "A comment about some concept"@en;
  .
  <http://catalog.org/doc/2019-01-11T12:00:00.000/some-concept> a prov:Entity;
    prov:generatedAtTime "2019-01-11T12:00:00.000"^^xsd:dateTime;
    dct:temporal [                                                                   #NEW
      dcmiperiod:start "2019-01-01"^^xsd:date;                                       #NEW
    ];                                                                               #NEW
    adms:status admsstatus:Completed;                                                #NEW
    prov:isPrimaryTopicOf <http://catalog.org/id/concept/some-concept>;
    prov:wasRevisionOf <http://catalog.org/doc/2019-01-09T12:00:00.000/some-concept>
  .
  <http://catalog.org/doc/2019-01-09T12:00:00.000/some-concept> a prov:Entity;
    prov:generatedAtTime "2019-01-09T12:00:00.000"^^xsd:dateTime;
    adms:status admsstatus:Withdrawn;                                                #NEW
    prov:isPrimaryTopicOf <http://catalog.org/id/concept/some-concept>;
    prov:wasRevisionOf <http://catalog.org/doc/2019-01-06T12:00:00.000/some-concept>
    prov:invalidatedAtTime "2019-01-12T12:00:00.000"^^xsd:dateTime;                  #NEW
  .
  <http://catalog.org/doc/2019-01-06T12:00:00.000/some-concept> a prov:Entity;
    prov:generatedAtTime "2019-01-06T12:00:00.000"^^xsd:dateTime;
    dct:temporal [
      dcmiperiod:start "2018-12-01"^^xsd:date;
      dcmiperiod:end "2019-01-01"^^xsd:date;                                         #NEW
    ];
    adms:status admsstatus:Completed;
    prov:isPrimaryTopicOf <http://catalog.org/id/concept/some-concept>;
    prov:wasRevisionOf <http://catalog.org/doc/2019-01-04T12:00:00.000/some-concept>
  .
  <http://catalog.org/doc/2019-01-04T12:00:00.000/some-concept> a prov:Entity;
    prov:generatedAtTime "2019-01-04T12:00:00.000"^^xsd:dateTime;
    adms:status admsstatus:Withdrawn;
    prov:isPrimaryTopicOf <http://catalog.org/id/concept/some-concept>;
    prov:wasRevisionOf <http://catalog.org/doc/2019-01-02T12:00:00.000/some-concept>
    prov:invalidatedAtTime "2019-01-07T12:00:00.000"^^xsd:dateTime;
  .
  <http://catalog.org/doc/2019-01-02T12:00:00.000/some-concept> a prov:Entity;
    prov:generatedAtTime "2019-01-02T12:00:00.000"^^xsd:dateTime;
    adms:status admsstatus:UnderDevelopment;
    prov:isPrimaryTopicOf <http://catalog.org/id/concept/some-concept>;
    prov:wasRevisionOf <http://catalog.org/doc/2019-01-01T12:00:00.000/some-concept>
    prov:invalidatedAtTime "2019-01-03T12:00:00.000"^^xsd:dateTime;
  .
  <http://catalog.org/doc/2019-01-01T12:00:00.000/some-concept> a prov:Entity;
    prov:generatedAtTime "2019-01-01T12:00:00.000"^^xsd:dateTime;
    adms:status admsstatus:UnderDevelopment;
    prov:isPrimaryTopicOf <http://catalog.org/id/concept/some-concept>
    prov:invalidatedAtTime "2019-01-02T12:00:00.000"^^xsd:dateTime;
  .
}
```

Three catalog records are affected by the valid-marking:
1. The catalog record that was valid from december first 2018 is no longer valid from januari first 2019;
2. The catalog record that was completed, but not marked valid is withdrawn (and deleted);
3. The catalog record that was made valid is set to completed, and made valid from januari first 2019.

#### TODO:

- Withdraw a catalog record: DELETE
- Mark a catalog record as invalid: POST (this is not a delete, because the prov:invalidatedAtTime is not set!) This also means that you cannot withdraw a catalog record that is already valid: not DELETE possible for such an item!
- Mark a catalog record as invalid will not "revalid" already existing catalog records.
- Making API calls for the catalog records themselves, instead of the topic of the catalog record.
