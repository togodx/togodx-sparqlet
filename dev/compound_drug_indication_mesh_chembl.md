# chembl mesh（山本, 守屋、信定） 作業中

* Top レベル ('C') はノードじゃ無いため URI もラベルもないので objectList 出力の Attribute は取れるもので最上位を出力

## Description

- Data sources
    - (More data sources description goes here..)
    - ChEMBL-RDF 28.0: http://ftp.ebi.ac.uk/pub/databases/chembl/ChEMBL-RDF/
    - Mesh 2021: ftp://ftp.nlm.nih.gov/online/mesh/rdf/
- Query
    - (More query details go here..)
    -  Input
        - ChEMBL ID
        - Mesh Tree Number
    - Output
        - Mesh term for ChEMBL drug indication

## Endpoint

https://integbio.jp/togosite/sparql

## `leaf`
- ChEMBL molecule - mesh D番号の対応
```sparql
PREFIX cco: <http://rdf.ebi.ac.uk/terms/chembl#> 
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX tree: <http://id.nlm.nih.gov/mesh/>
PREFIX meshv: <http://id.nlm.nih.gov/mesh/vocab#>
SELECT DISTINCT?child ?child_label ?parent ?parent_label
FROM <http://rdf.integbio.jp/dataset/togosite/chembl>
FROM <http://rdf.integbio.jp/dataset/togosite/mesh>
WHERE {
  ?molecule cco:chemblId ?child ;
            rdfs:label ?child_label ;
            cco:hasDrugIndication [
    a cco:DrugIndication ;
    cco:hasMesh ?parent_ori
  ] .
  BIND(IRI(REPLACE(STR(?parent_ori), "http://identifiers.org/mesh/","http://id.nlm.nih.gov/mesh/")) AS ?parent)
  ?parent meshv:treeNumber/meshv:parentTreeNumber* ?tree .
  ?tree ^meshv:treeNumber/rdfs:label ?parent_label .
}
```
## `graph`
- Mesh の親子関係
```sparql
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX meshv: <http://id.nlm.nih.gov/mesh/vocab#>
PREFIX tree: <http://id.nlm.nih.gov/mesh/>
SELECT DISTINCT ?mesh ?parent ?parent_label (SAMPLE(?child_tree) AS ?child)
FROM <http://rdf.integbio.jp/dataset/togosite/mesh>
WHERE {
  ?parent a meshv:TreeNumber .
  MINUS { 
    ?parent meshv:parentTreeNumber ?parent_tree .
  }
  FILTER (CONTAINS(STR(?parent),"mesh/C"))
   ?parent ^meshv:treeNumber/rdfs:label ?parent_label .
   ?mesh meshv:treeNumber/meshv:parentTreeNumber* ?parent .
   OPTIONAL {
     ?child_tree meshv:parentTreeNumber ?parent .
   }
}
```
# `return`
```javascript
({root, leaf, graph, allLeaf}) => {
  const idPrefix = "http://purl.uniprot.org/uniprot/";
  const categoryPrefix = "http://purl.obolibrary.org/obo/";
  const withoutId = "unclassified";
  
  let tree = [
    {
      id: root,
      root: true
    },{
      id: withoutId,
      label: "Unclassified",
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
    withAnnotation[d.child.value] = true;
    tree.push({
      id: d.child.value.replace(idPrefix, ""),
      label: d.child_label.value,
      leaf: true,
      parent: d.parent.value.replace(categoryPrefix, "")
    })
  })
  // アノテーション無し要素
  allLeaf.results.bindings.map(d => {
    if (!withAnnotation[d.leaf.value]) {
      tree.push({
        id: d.leaf.value.replace(idPrefix, ""),
        label: d.leaf_label.value,
        leaf: true,
        parent: withoutId
      });
    }
  })
  
  return tree;	
}
```