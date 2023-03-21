# ChEBI reactome-pathway classification（申）

- 
  - 
  
## Description

- Data sources
    - 
    - 
        - 
- Query
    - Input
        - 
    - Output
        - 

## Endpoint
https://integbio.jp/togosite/sparql

## `leaf`
- pathway と UniProt のアノテーション関係
- 産物と制御
  - 制御: reaction ^biopax:controlled/biopax:controller control-component
  - 産物: reaction biopax:left|biopax:right|biopax:product product-component
```sparql
PREFIX biopax: <http://www.biopax.org/release/biopax-level3.owl#>
PREFIX xsd: <http://www.w3.org/2001/XMLSchema#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX obo: <http://purl.obolibrary.org/obo/>
SELECT DISTINCT ?parent ?child ?child_path ?child_label
FROM <http://rdf.integbio.jp/dataset/togosite/reactome>
FROM <http://rdf.integbio.jp/dataset/togosite/chebi>
WHERE {
  ?top_path a biopax:Pathway ;
            biopax:organism ?org .
  ?org biopax:name "Homo sapiens"^^xsd:string .
  MINUS { [] biopax:pathwayComponent ?top_path . }
  ?top_path biopax:pathwayComponent+ ?path .
  ?path a biopax:Pathway ;
          biopax:displayName ?parent_label ;
          biopax:xref [
            biopax:db "Reactome"^^xsd:string ;
                      biopax:id ?parent ;
          ] ;
          biopax:pathwayComponent ?child_path .
  ?child_path a biopax:Pathway ;
                biopax:displayName ?child_label ;
                biopax:xref [
                  biopax:db "Reactome"^^xsd:string ;
                            biopax:id ?child 
                ] .

}
```

## `graph`
- pathway の親子関係
```sparql
PREFIX biopax: <http://www.biopax.org/release/biopax-level3.owl#>
PREFIX xsd: <http://www.w3.org/2001/XMLSchema#>
SELECT DISTINCT ?parent ?child ?parent_label ?child_label
FROM <http://rdf.integbio.jp/dataset/togosite/reactome>
WHERE {
  ?top_path a biopax:Pathway ;
            biopax:organism ?org .
  ?org biopax:name "Homo sapiens"^^xsd:string .
  MINUS { [] biopax:pathwayComponent ?top_path . }
  ?top_path biopax:pathwayComponent* ?parent_path .
  ?parent_path a biopax:Pathway ;
                 biopax:displayName ?parent_label ;
                 biopax:xref [
                   biopax:db "Reactome"^^xsd:string ;
                             biopax:id ?parent 
                 ] ;
                 biopax:pathwayComponent ?child_path .
  ?child_path a biopax:Pathway ;
                biopax:displayName ?child_label ;
                biopax:xref [
                  biopax:db "Reactome"^^xsd:string ;
                            biopax:id ?child 
                ] .
}
```

## `top`
- ヒト pathway の top 階層
```sparql
PREFIX biopax: <http://www.biopax.org/release/biopax-level3.owl#>
PREFIX xsd: <http://www.w3.org/2001/XMLSchema#>
SELECT DISTINCT ?top ?top_label
FROM <http://rdf.integbio.jp/dataset/togosite/reactome>
WHERE {
  ?top_path a biopax:Pathway ;
            biopax:pathwayComponent ?child_path .
  MINUS { [] biopax:pathwayComponent ?top_path . }
  ?top_path biopax:organism ?org ;
            biopax:displayName ?top_label ;
            biopax:xref [
              biopax:db "Reactome"^^xsd:string ;
              biopax:id ?top ;
            ] .
  ?org biopax:name "Homo sapiens"^^xsd:string .
}
```

## `return`
- 整形
```javascript
({top, leaf, graph}) => {
  const idPrefix = "CHEBI:";
  const withoutId = "unclassified";

  let tree = [
    {
      id: "root",
      label: "root node",
      root: true
    }
  /*  ,{
      id: withoutId,
      label: "without annotation",
      parent: "root"
    } */
  ];
  
  let check = {};
  leaf.results.bindings.map(d => {
    check[d.child_path.value] = true;
  })
  
  let withAnnotation = {};
  // トップとルートの関係
  top.results.bindings.map(d => {
    tree.push({
      id: d.top.value.replace("R-HSA-", "R_HSA_"),
      label: d.top_label.value,
      parent: "root"
    })
  })
  // 親子関係
  graph.results.bindings.map(d => {
    if (check[d.parent.value]) {  // proteome check
      tree.push({
        id: d.child.value.replace("R-HSA-", "R_HSA_"),
        label: d.child_label.value,
        
        parent: d.parent.value.replace("R-HSA-", "R_HSA_")
      })
    } else {
      tree.push({
        id: d.child.value.replace("R-HSA-", "R_HSA_"),
        label: d.child_label.value,
        leaf: true,
        parent: d.parent.value.replace("R-HSA-", "R_HSA_")
      })
    }
  })
  /*
  // アノテーション関係
  leaf.results.bindings.map(d => {
    
    tree.push({
      id: d.child.value,
      label: d.child_label.value,
      leaf: true,
      parent: d.parent.value.replace("R-HSA-", "R_HSA_")
    })
  })
  */
/*  // アノテーション無し要素
  allLeaf.results.bindings.map(d => {
    if (!withAnnotation[d.leaf.value]) {
      tree.push({
        id: d.leaf.value.replace(idPrefix, ""),
        label: d.leaf_label.value,
        leaf: true,
        parent: withoutId
      });
    }
  }) */
  
  return tree;
}
```