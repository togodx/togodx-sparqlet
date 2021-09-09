# UniProt reactome-pathway classification（守屋）

## Description

- Data sources
    - Reactome-classified pathways for each protein
    - This item based on the data of Reactome Version 71 (02, December 2019).
       - The latest data can be obtained from the URL below. https://reactome.org/download-data
- Query
    - Input
        - Reactome ID, Uniprot ID
    - Output
        - The number of Uniprot entries included in each pathway in Reactome
        - If a Uniprot id is entered, it returns the pathway to which the protein belongs

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
SELECT DISTINCT ?parent ?child ?parent_label
FROM <http://rdf.integbio.jp/dataset/togosite/reactome>
WHERE {
  ?target_path a biopax:Pathway ;
        biopax:displayName ?parent_label ;
        biopax:xref [
          biopax:db "Reactome"^^xsd:string ;
          biopax:id ?parent ;
        ] ;
        biopax:pathwayComponent ?reaction .
  ?reaction a biopax:BiochemicalReaction .
  ?reaction biopax:left|biopax:right|biopax:product|^biopax:controlled/biopax:controller ?component .
  ?component (biopax:memberPhysicalEntity|biopax:component)*/biopax:entityReference/biopax:xref [
    biopax:db "UniProt"^^xsd:string ;
    biopax:id ?child
  ] .
}
```

## `graph`
- pathway の親子関係
```sparql
PREFIX biopax: <http://www.biopax.org/release/biopax-level3.owl#>
PREFIX xsd: <http://www.w3.org/2001/XMLSchema#>
SELECT DISTINCT ?parent ?child ?parent_label
FROM <http://rdf.integbio.jp/dataset/togosite/reactome>
WHERE {
  ?parent_path a biopax:Pathway ;
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
          biopax:id ?child ;
        ] .
}
```

## `allLeaf`
- 全 UniProt (without annotation 用)
```sparql
PREFIX up: <http://purl.uniprot.org/core/>
PREFIX taxon: <http://purl.uniprot.org/taxonomy/>
SELECT DISTINCT ?leaf ?leaf_label
FROM <http://rdf.integbio.jp/dataset/togosite/uniprot>
FROM <http://rdf.integbio.jp/dataset/togosite/go>
WHERE {
  ?leaf a up:Protein ;
        up:organism taxon:9606 ;
        up:mnemonic ?leaf_label ;
        up:proteome ?proteome .
  FILTER(REGEX(STR(?proteome), "UP000005640"))
}
```

## `return`
- 整形
```javascript
({mode, data})=>{
  let idVarName = "uniprot";
  if (mode == "objectList") return data.results.bindings.map(d=>{
    return {
      id: d[idVarName].value,
      attribute: {
        categoryId: d.category.value,
        label: d.label.value
      }
    }
  });
  if (mode == "idList") return data.results.bindings.map(d=>d[idVarName].value);

  return data.results.bindings.map(d=>{
    let hasChild = false;
    if (d.child) hasChild = true;
    return {
      categoryId: d.category.value, 
      label: d.label.value,
      count: Number(d.count.value),
      hasChild: hasChild
    };
  });	
}
```