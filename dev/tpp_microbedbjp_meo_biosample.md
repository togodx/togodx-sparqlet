# TPP MicrobeDB.jp MEO annotation

## Parameters

* `root`
  * default: MEO_0001240
  * example: MEO_0001240, MEO_0000817 (component, environment)

## Endpoint

https://microbedb.jp/sparql

## `leaf`
```sparql
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX sio: <http://semanticscience.org/resource/>
PREFIX skos: <http://www.w3.org/2004/02/skos/core#>
PREFIX meo: <http://purl.jp/bio/11/meo/>
PREFIX obo: <http://purl.obolibrary.org/obo/>
PREFIX tax: <http://ddbj.nig.ac.jp/ontologies/taxonomy/>
PREFIX mdbv: <http://purl.jp/bio/11/mdbv#>
SELECT DISTINCT ?child ?child_label ?parent
WHERE {
  ?child a sio:SIO_001050 ;
          obo:RO_0002162 ?taxon ;
          sio:SIO_000255/sio:SIO_000255 [
            a mdbv:EnvAnnotation ;
            sio:SIO_000671 ?parent
          ] .
  ?parent rdfs:isDefinedBy meo: ;
          rdfs:subClassOf* meo:{{root}} .
  ?taxon rdfs:label ?child_label .
}
```

## `graph`
```sparql
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX meo: <http://purl.jp/bio/11/meo/>
SELECT DISTINCT ?child ?child_label ?parent ?parent_label
WHERE {
  ?parent rdfs:subClassOf* meo:{{root}} ;
          rdfs:label ?parent_label .
  ?child rdfs:subClassOf ?parent ;
       rdfs:label ?child_label .
}
```

## `return`
```javascript
({root, leaf, graph}) => {
  const idPrefix = "http://identifiers.org/biosample/";
  const categoryPrefix = "http://purl.jp/bio/11/meo/";
  
  let tree = [
    {
      id: root,
      root: true
    }
  ];

  // 親子関係
  graph.results.bindings.map(d => {
    tree.push({
      id: d.child.value.replace(categoryPrefix, ""),
      label: d.child_label.value,
      parent: d.parent.value.replace(categoryPrefix, "")
    })
    if (d.parent.value.replace(categoryPrefix, "") == root && !tree[0].label) tree[0].label = d.parent_label.value; // root の label 挿入
  })
  // アノテーション関係
  leaf.results.bindings.map(d => {
    tree.push({
      id: d.child.value.replace(idPrefix, ""),
      label: d.child_label.value,
      leaf: true,
      parent: d.parent.value.replace(categoryPrefix, "")
    })
  })
  
  return tree;
}
```