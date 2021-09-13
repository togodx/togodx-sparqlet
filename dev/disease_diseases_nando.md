# 日本の難病疾患のカテゴリフィルター（NANDO階層利用）(高月)作業中

## Description

- Data sources
    - Nanbyo Disease Ontology (NANDO):[http://nanbyodata.jp/ontology/nando](http://nanbyodata.jp/ontology/nando)
- Input/Output
     -  Input
        - NANDO ID
    - Output
        - NANDO category
- Supplementary information


## Endpoint

https://integbio.jp/togosite/sparql

## `data`

```sparql
PREFIX nando: <http://nanbyodata.jp/ontology/NANDO_>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>

#SELECT DISTINCT ?id ?label ?parent
SELECT ?tree ?id ?parent ?label SAMPLE(?child) AS ?child
FROM <http://rdf.integbio.jp/dataset/togosite/nando>
WHERE {
#  VALUES ?disease_root { nando: }
  
  ?id rdfs:label ?label.
  FILTER (lang(?label) = "en")
  ?id rdfs:subClassOf* ?parent.
   
  # ?diseases_rootのエントリにはparentが存在しないのでOPTIONALが必要
  OPTIONAL {
    ?id rdfs:subClassOf ?parent.
  }
  
  # 中間ノードの場合は?childに値が存在し、leafノードの場合は?childは存在しないのでOPTIONALが必要
  OPTIONAL {
    ?id ^rdfs:subClassOf ?child.
  }

  FILTER(?parent == NULL)
}
GROUP BY ?tree ?id ?parent ?label 
```


## `return`

```javascript
({data}) => {
  const idPrefix = "http://nanbyodata.jp/ontology/NANDO_";
  let tree = [
    {
      id: "root",
      label: "root node",
      root: true
    }
  ];
  data.results.bindings.forEach(d => {
    tree.push({
      id: d.id.value.replace(idPrefix, ""),
      label: d.label.value,
      leaf: (d.child == undefined ? true : false),
      parent: (d.parent == undefined ? "root" :  d.parent.value.replace(idPrefix, ""))
    });
  });
  return tree;
};
```

## MEMO
-Author
 - Takatsuki