# Diseaseカテゴリフィルタ(HPO階層利用)(三橋・高月)(作業中）

## Description

- Data sources
    -  [Human Phenotype Ontology (HPO)](https://hpo.jax.org/app/) 
- Query
    - Input
        - HPO id
    - Output
        -  [Phenotypic abnormality (HP:0000118)](https://hpo.jax.org/app/browse/term/HP:0000118)  and its subcategories of HPO

## Endpoint

https://integbio.jp/togosite/sparql

## `data`
```sparql

PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX dct: <http://purl.org/dc/terms/>
PREFIX obo: <http://purl.obolibrary.org/obo/>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX owl: <http://www.w3.org/2002/07/owl#>
SELECT DISTINCT ?hp ?label ?parent ?child
FROM <http://rdf.integbio.jp/dataset/togosite/hpo>
WHERE {
  #root nodeはHP_000118
  VALUES ?root {  obo:HP_0000118  }    
  ?hp rdfs:label ?label.
  ?hp rdfs:subClassOf* ?parent.
  ?hp rdf:type owl:Class.  
  # HP以外のIDも登録されているため、HPに限定するためにフィルターを追加
  FILTER regex(str(?hp), "http://purl.obolibrary.org/obo/HP_" )
  # 中間ノードの場合は？parentに値が存在し、leafノードの場合は?parentは存在しないのでOPTIONALが必要
      OPTIONAL {
        ?hp ^rdfs:subClassOf ?child.
        }
 } 
 GROUP BY ?hp ?parent ?label
             
```

