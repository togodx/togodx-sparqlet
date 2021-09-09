# WIP: ChEBI Application での分類 （川島、建石） Server対応中未完


## Parameters

* `root` (type: obo:CHEBI_) (Req.)
  * default: 33232
  * example: 33232 (Application), 52217 (pharmaceutical), 64047 (food additive)
  


## Endpoint
https://integbio.jp/togosite/sparql

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
  #test
  VALUES ?compound { CHEBI:18012  CHEBI:27732 CHEBI:17594 CHEBI:16866 CHEBI:46195 CHEBI:62867}
      
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

## `return`

```javascript

({root, leaf, graph}) => {
  const idPrefix = "http://purl.obolibrary.org/obo/CHEBI_";
  const categoryPrefix = "http://purl.obolibrary.org/obo/CHEBI_";
  const withoutId = "without_annotation";
  
  let tree = [
    {
      id: root,
      root: true
    },{
      id: withoutId,
      label: "without annotation",
      parent: root
    }
  ];

  let withAnnotation = {};
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
    withAnnotation[d.compound.value] = true;
    tree.push({
      id: d.compound.value.replace(idPrefix, ""),
      label: d.compound_label.value,
      leaf: true,
      parent: d.application.value.replace(categoryPrefix, "")
    })
  })
  // アノテーション無し要素
  //allLeaf.results.bindings.map(d => {
    //if (!withAnnotation[d.leaf.value]) {
      //tree.push({
        //id: d.leaf.value.replace(idPrefix, ""),
        //label: d.leaf_label.value,
        //leaf: true,
        //parent: withoutId
      //});
    //}
  //})
  
  return tree;	
}

```