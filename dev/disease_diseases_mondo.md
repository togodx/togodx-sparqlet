# Diseaseカテゴリフィルタ(Mondo階層利用)(三橋)

## Description

- Data sources
    -  [Mondo Disease Ontology (Mondo) ](https://mondo.monarchinitiative.org/) 
- Query
    - Input
        - Mondo id
    - Output
        -  [Disease and disorder (MONDO_0000001)](https://monarchinitiative.org/disease/MONDO:0000001) and its subcategories of Mondo

## Parameters

* `root` (type: Mondo) (Req.)
  * default: 0000001
  * example: GO_0008150 (biological process), GO_0005575 (cellular component), GO_0003674 (molecular function)

## Endpoint

https://integbio.jp/togosite/sparql

## `data`
```sparql
PREFIX dct: <http://purl.org/dc/terms/>
PREFIX mondo: <http://purl.obolibrary.org/obo/>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX owl: <http://www.w3.org/2002/07/owl#>

SELECT DISTINCT ?mondo ?label
FROM <http://rdf.integbio.jp/dataset/togosite/mondo>
WHERE {
  ?mondo rdfs:subClassOf+ mondo:MONDO_0000001 .
  ?mondo rdfs:label ?label.
}
```