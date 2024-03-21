# chembl mesh（山本、守屋、信定、建石）
* Server対応済み
* MeSH の D番号を node ID として使うと、情報が減って DAG がループしてしまうので、Tree番号をIDとする

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

{{SPARQLIST_TOGODX_SPARQL}}

## `chemblHasMesh`
```sparql
PREFIX chembl: <http://rdf.ebi.ac.uk/terms/chembl#> 
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>

SELECT DISTINCT ?chembl_id ?chembl_label ?mesh
FROM <http://rdf.integbio.jp/dataset/togosite/chembl>
WHERE {
  ?molecule chembl:chemblId ?chembl_id ;
      rdfs:label ?chembl_label ;
      chembl:hasDrugIndication/chembl:hasMesh ?mesh .
}
```

## `meshTree`
```sparql
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX mesh: <http://id.nlm.nih.gov/mesh/>
PREFIX meshv: <http://id.nlm.nih.gov/mesh/vocab#>

SELECT DISTINCT ?mesh_id ?mesh_label ?node ?parent
FROM <http://rdf.integbio.jp/dataset/togosite/mesh>
WHERE {
  # MeSH Treeの Diseases[C] 以下を取得
  # See https://meshb.nlm.nih.gov/treeView
  FILTER (regex(str(?tree), "C"))
  ?node meshv:parentTreeNumber* ?tree .
  OPTIONAL {
    ?node meshv:parentTreeNumber ?parent  .
  }

  ?mesh_id meshv:treeNumber ?node ;
      rdfs:label ?mesh_label .
  FILTER(lang(?mesh_label) = "en")
}
```

# `return`
```javascript
({ chemblHasMesh, meshTree }) => {
  const idPrefix = "http://id.nlm.nih.gov/mesh/";
  const idMeshPrefix = "http://identifiers.org/mesh/";

  let tree = [
    {
      id: "root",
      label: "root node",
      root: true
    }
  ];
  let mesh2tree = {};
  
  meshTree.results.bindings.forEach((d) => {
    const tree_id = d.node.value.replace(idPrefix, "");
    const mesh_id = d.mesh_id.value.replace(idPrefix, "");
    tree.push({
      id: tree_id,
      label: d.mesh_label.value,
      parent: d.parent ? d.parent.value.replace(idPrefix, "") : "root"
    }); 
    if (! mesh2tree[mesh_id]) mesh2tree[mesh_id] = [];
    mesh2tree[mesh_id].push(tree_id);
  });

  chemblHasMesh.results.bindings.forEach((d) => {
    const mesh_id = d.mesh.value.replace(idMeshPrefix, "");
    if (mesh2tree[mesh_id]) {
      for (let tree_id of mesh2tree[mesh_id]) {
        tree.push({
          id: d.chembl_id.value,
          label: d.chembl_label.value,
          leaf: true,
          parent: tree_id
        });
      }
    }
  });

  return tree;
}
```
