# MONDOにおける、関連付けされているDBのカウント(高月)作業中

## Description

- Data sources
     - Mondo disease ontlogy: [https://mondo.monarchinitiative.org/](https://mondo.monarchinitiative.org/)
- Input/Output
     -  Input
        - MONDO ID
    - Output
        - Related Databases category
- Supplementary information
     - The data counted the number of associations for each of the approximate 70 disease DBs associated with the lexical data MONDO, which is an exhaustive collection of disease names.
     - 網羅的に疾患名を集めている語彙データMONDOに関連づけられているおおよそ70の各疾患DBに対して、関連付けされた数をカウントしたデータ。
  
## Endpoint

https://integbio.jp/togosite/sparql

## `data`

```sparql
PREFIX mondo: <http://purl.obolibrary.org/obo/MONDO_>
PREFIX obo: <http://purl.obolibrary.org/obo/>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX oboinowl: <http://www.geneontology.org/formats/oboInOwl#>


SELECT DISTINCT ?mondo ?name
FROM <http://rdf.integbio.jp/dataset/togosite/mondo>
WHERE {
  ?mondo oboinowl:hasDbXref ?id .
  BIND (strbefore(str(?id), ":") AS ?name)  
  FILTER(!STRSTARTS(str(?id), "http"))
  FILTER regex(str(?mondo), "http://purl.obolibrary.org/obo/MONDO_" )
}

```
## `return`
- 整形
```javascript

```

## MEMO
-Author
 - Takatsuki