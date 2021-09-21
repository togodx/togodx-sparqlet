# Diseases in Mondo (三橋・高月) （作業中）

## Description

- Data sources
    -  [Mondo Disease Ontology (Mondo) ](https://mondo.monarchinitiative.org/) 
- Query
    - Input
        - Mondo id
    - Output
        -  [Disease and disorder (MONDO_0000001)](https://monarchinitiative.org/disease/MONDO:0000001) and its subcategories of Mondo

## Endpoint

https://integbio.jp/togosite/sparql

## `data`
```sparql
PREFIX dct: <http://purl.org/dc/terms/>
PREFIX mondo: <http://purl.obolibrary.org/obo/>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX owl: <http://www.w3.org/2002/07/owl#>

#SELECT COUNT(DISTINCT ?mondo) 
SELECT DISTINCT ?mondo ?label ?parent ?child
FROM <http://rdf.integbio.jp/dataset/togosite/mondo>
WHERE {
  #root nodeはMONDO_0000001
  VALUES ?root {mondo:MONDO_0000001}
  
  ?mondo rdfs:subClassOf+ mondo:MONDO_0000001 .
  ?mondo rdfs:label ?label.
  ?mondo rdfs:subClassOf ?parent.
  FILTER(!STRSTARTS(str(?parent), "nodeID:"))
    # 中間ノードの場合は?parentに値が存在し、leafノードの場合は?parentは存在しないのでOPTIONALが必要
  OPTIONAL {
    ?mondo ^rdfs:subClassOf ?child.
      }
   }
GROUP BY ?mondo ?parent ?label 
```