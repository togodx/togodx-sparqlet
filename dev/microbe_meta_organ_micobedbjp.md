# Detected organs of human metagenome 16S rRNA analysis from MicrobeDB.jp

## Endpoint

https://microbedb.jp/sparql

## `leaf`
```sparql
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX sio: <http://semanticscience.org/resource/>
PREFIX meo: <http://purl.jp/bio/11/meo/>
PREFIX mdbv: <http://purl.jp/bio/11/mdbv#>
PREFIX obo: <http://purl.obolibrary.org/obo/>
PREFIX tax: <http://ddbj.nig.ac.jp/ontologies/taxonomy/>
SELECT DISTINCT ?child ?child_label ?parent
WHERE {
  VALUES ?rank { tax:Genus tax:NoRank }
  [] a sio:SIO_001050 ;
     sio:SIO_000008 [
       a mdbv:HostName;
       sio:SIO_000300 "Homo sapiens"
     ] ;
     mdbv:has_analysis [
       a mdbv:TaxonomicAnnotationOfMicrobiomeBasedOn16SrRNA ;
       sio:SIO_000216 [
         mdbv:has_key ?child
       ] 
     ] ;
     sio:SIO_000255/sio:SIO_000255 [
       a mdbv:EnvAnnotation ; 
       sio:SIO_000671 ?parent
     ] .
  ?parent rdfs:isDefinedBy	meo: ;
       rdfs:subClassOf* meo:MEO_0000175 . # foundamental organ
  ?child rdfs:label ?child_label ;
         tax:rank ?rank .
}
```

## `graph`
```sparql
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX meo: <http://purl.jp/bio/11/meo/>
SELECT DISTINCT ?child ?child_label ?parent ?parent_label
WHERE {
  ?parent rdfs:subClassOf* meo:MEO_0000175 .
  ?child rdfs:subClassOf ?parent ;
       rdfs:label ?child_label .
}
```

## `return`
```javascript
({leaf, graph}) => {
  const idPrefix = "http://identifiers.org/taxonomy/";
  const categoryPrefix = "http://purl.jp/bio/11/meo/";

  let tree = [
    {
      id: "MEO_0000175",
      label: "foundamental organ",
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