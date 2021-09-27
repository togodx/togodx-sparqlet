# ChEBI classification での分類 （川島、建石） 
- アノテーションがないケースを数えていない（もとのまま）
## Parameters

* `root`
  * default: 33232
  * example: 33232 (Application type), 24432 (Biological role), 51086 (Chemical role)

## Endpoint
https://integbio.jp/togosite/sparql

## `graph`
- Application の親子関係

```sparql
PREFIX obo: <http://purl.obolibrary.org/obo/>
PREFIX owl: <http://www.w3.org/2002/07/owl#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>

SELECT DISTINCT ?parent ?child ?parent_label ?child_label
FROM <http://rdf.integbio.jp/dataset/togosite/chebi>
WHERE 
{
      
  ?r a owl:Restriction ;
    owl:onProperty obo:RO_0000087 ;
    owl:someValuesFrom ?child .
  ?child rdfs:subClassOf* obo:CHEBI_{{root}} .
  ?child rdfs:subClassOf ?parent .
  ?child rdfs:label ?child_label .
  ?parent rdfs:label ?parent_label .
  
}
```

## `leaf`
- ChEBI Compound  のアノテーション

```sparql
PREFIX obo: <http://purl.obolibrary.org/obo/>
PREFIX owl: <http://www.w3.org/2002/07/owl#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>

SELECT distinct ?compound ?category (GROUP_CONCAT(DISTINCT ?label; SEPARATOR = ", ") as ?compound_label)                                
FROM <http://rdf.integbio.jp/dataset/togosite/chebi>
WHERE 
{     
  ?compound a owl:Class ;
    rdfs:label ?label ;
    rdfs:subClassOf ?r .
  ?r a owl:Restriction ;
    owl:onProperty obo:RO_0000087 ;
    owl:someValuesFrom ?application .
  ?category rdfs:subClassOf*  obo:CHEBI_{{root}} .
}
```

## `return`

```javascript
({root, leaf, graph}) => {
  const idPrefix = "http://purl.obolibrary.org/obo/CHEBI_";
  const categoryPrefix = "http://purl.obolibrary.org/obo/CHEBI_";
  
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
      id: d.compound.value.replace(idPrefix, ""),
      label: d.compound_label.value,
      leaf: true,
      parent: d.category.value.replace(categoryPrefix, "")
    })
  })
  
  return tree;	
}
```