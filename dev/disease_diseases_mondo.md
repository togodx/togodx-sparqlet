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
SELECT ?mondo ?label ?parent SAMPLE(?child) AS ?child
FROM <http://rdf.integbio.jp/dataset/togosite/mondo>
WHERE {
  #root nodeはMONDO_0000001
  VALUES ?root { mondo:MONDO_0000001 }
  
  ?mondo rdfs:subClassOf+ ?root .
  ?mondo rdfs:label ?label.
  ?mondo rdfs:subClassOf ?parent.
  FILTER(!STRSTARTS(str(?parent), "nodeID:"))
  # 中間ノードの場合は?parentに値が存在し、leafノードの場合は?parentは存在しないのでOPTIONALが必要
  OPTIONAL {
    ?mondo ^rdfs:subClassOf ?child.
  }
}
GROUP BY ?mondo ?parent ?label 
#limit 100
```
## `return`

```javascript
({data}) => {
  const idPrefix = "http://purl.obolibrary.org/obo/MONDO_";
  let tree = [
    {
      id: "0000001",
      label: "Disease and disorder",
      root: true
    }
  ];
  data.results.bindings.forEach(d => {
    tree.push({
      id: d.mondo.value.replace(idPrefix, ""),
      label: d.label.value,
      leaf: (d.child == undefined ? true : false),
      parent: d.parent.value.replace(idPrefix, "")
    });
  });
  return tree;
};
```

## MEMO
-Author
 - Takatsuki
 - Mitsuhashi