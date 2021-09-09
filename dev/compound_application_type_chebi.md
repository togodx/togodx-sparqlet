# WIP: ChEBI Application での分類 （川島、建石） Server対応中未完

- backend SPARQLet

## Description

- Data sources
    -  [Chemical Entities of Biological Interest (ChEBI) ](https://www.ebi.ac.uk/chebi/) 
- Query
    - Input
        - ChEBI id (number) for chemical compound(s)
    - Output
        -  ChEBI id (number) for application type(s) (subcategories of [Application (CHEBI:33232)](https://www.ebi.ac.uk/chebi/searchId.do?chebiId=CHEBI:33232) ) corresponding to the compound(s)
- Supplementary Information
	-  The classification of compounds according to their application, defined in ChEBI ontology.
	- ChEBI Ontologyに定義された用途による、	化合物の分類です。

## Parameters

* `root` (type: CHEBI) (Req.)
  * default: 33232
  * example: 33232 (Application), 52217 (pharmaceutical), 64047 (food additive)
* `queryIds` 化合物のID（数字のみ）またはIDのリスト (テスト用、後で消す）
  * default: 18012, 27732, 17594, 16866, 46195, 62867
  * example: 18012, 27732, 17594, 16866, 46195, 62867     

## Endpoint
https://integbio.jp/togosite/sparql

## `queryArray`
- ユーザが指定した ID リストを配列に分割
- テスト用、後で消す

```javascript
({queryIds}) => {
  queryIds = queryIds.replace(/,/g," ")
   if (queryIds.match(/[^\s]/))  return queryIds.split(/\s+/);
  return false;
}
```




## `leaf`
- ChEBI Compound  のアノテーション
```sparql
PREFIX obo: <http://purl.obolibrary.org/obo/>
PREFIX owl: <http://www.w3.org/2002/07/owl#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX CHEBI: <http://purl.obolibrary.org/obo/CHEBI_>

SELECT distinct ?compound GROUP_CONCAT(DISTINCT ?label; SEPARATOR = ", ") as ?label 
                          ?application                                      
                          GROUP_CONCAT(DISTINCT ?application_label; SEPARATOR = ", ") AS ?application_label
FROM <http://rdf.integbio.jp/dataset/togosite/chebi>
WHERE 
{
  VALUES ?compound { {{#each queryArray}} CHEBI:{{this}} {{/each}} }
      
  ?compound a owl:Class ;
    rdfs:label ?label ;
    rdfs:subClassOf ?r .
  ?r a owl:Restriction ;
    owl:onProperty obo:RO_0000087 ;
    owl:someValuesFrom ?application .
  ?application rdfs:subClassOf* obo:CHEBI_{{root}}.  
  ?application rdfs:label ?application_label .
  
}
```

## `graph`
- Application の親子関係
```sparql
PREFIX obo: <http://purl.obolibrary.org/obo/>
PREFIX owl: <http://www.w3.org/2002/07/owl#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX CHEBI: <http://purl.obolibrary.org/obo/CHEBI_>

SELECT DISTINCT ?parent ?child ?parent_label ?child_label
FROM <http://rdf.integbio.jp/dataset/togosite/chebi>
WHERE 
{
      
  ?r a owl:Restriction ;
    owl:onProperty obo:RO_0000087 ;
    owl:someValuesFrom ?child .
  ?child rdfs:subClassOf* obo:CHEBI_{{root}}.
  ?child rdfs:subClassOf ?parent.
  ?child rdfs:label ?child_label .
  ?parent rdfs:label ?parent_label .
  
}```

