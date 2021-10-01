# chembl mesh（山本、守屋、信定）
Server対応済み

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

SELECT DISTINCT ?mesh_id ?mesh_label ?parent_mesh_id
FROM <http://rdf.integbio.jp/dataset/togosite/mesh>
WHERE {
  # MeSH TreeのRoot(Diseases[C]) のURI もラベルもないので、その下の階層(Infections[C01],...)のDescriptor(D007239)を列挙する
  # See https://meshb.nlm.nih.gov/treeView
  VALUES ?diseases { mesh:D007239 mesh:D009369 mesh:D009140 mesh:D004066 mesh:D009057 mesh:D012140 mesh:D010038 mesh:D009422 mesh:D005128 mesh:D052801 mesh:D005261 mesh:D002318 mesh:D006425 mesh:D009358 mesh:D017437 mesh:D009750 mesh:D004700 mesh:D007154 mesh:D007280 mesh:D000820 mesh:D013568 mesh:D009784 }
  ?diseases meshv:treeNumber ?tree .

  ?node meshv:parentTreeNumber* ?tree .
  OPTIONAL {
    ?node meshv:parentTreeNumber ?parent  .
    ?parent_mesh_id meshv:treeNumber ?parent .
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

  let tree = [
    {
      id: "root",
      label: "root node",
      root: true
    }
  ];

  meshTree.results.bindings.forEach((d) => {
    tree.push({
      id: d.mesh_id.value.replace(idPrefix, ""),
      label: d.mesh_label.value,
      parent: d.parent_mesh_id ? d.parent_mesh_id.value.replace(idPrefix, "") : "root"
    });
  });

  chemblHasMesh.results.bindings.map((d) => {
    tree.push({
      id: d.chembl_id.value,
      label: d.chembl_label.value,
      leaf: true,
      parent: d.mesh.value.replace('http://identifiers.org/mesh/', '')
    });
  });

  return tree;
}
```
