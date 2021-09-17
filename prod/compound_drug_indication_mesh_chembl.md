# chembl mesh（山本, 守屋、信定）
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

## `leaf`
- ChEMBL molecule - mesh D番号の対応
```sparql
PREFIX cco: <http://rdf.ebi.ac.uk/terms/chembl#> 
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX tree: <http://id.nlm.nih.gov/mesh/>
PREFIX meshv: <http://id.nlm.nih.gov/mesh/vocab#>
SELECT DISTINCT?child ?child_label ?parent 
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
}
```
## `graph`
- Mesh の親子関係
```sparql
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX mesh: <http://id.nlm.nih.gov/mesh/>
PREFIX meshv: <http://id.nlm.nih.gov/mesh/vocab#>
PREFIX tree: <http://id.nlm.nih.gov/mesh/>

SELECT ?tree ?id ?parent ?label SAMPLE(?tree_child) AS ?tree_child
FROM <http://rdf.integbio.jp/dataset/togosite/mesh>
WHERE {
  # MeSH TreeのRoot(Diseases[C]) のURI もラベルもないので、その下の階層(Infections[C01],...)のDescriptor(D007239)を列挙する
  # See https://meshb.nlm.nih.gov/treeView
  VALUES ?diseases_root { mesh:D007239 mesh:D009369 mesh:D009140 mesh:D004066 mesh:D009057 mesh:D012140 mesh:D010038 mesh:D009422 mesh:D005128 mesh:D052801 mesh:D005261 mesh:D002318 mesh:D006425 mesh:D009358 mesh:D017437 mesh:D009750 mesh:D004700 mesh:D0071154 mesh:D007280 mesh:D000820 mesh:D013568 mesh:D009784 }
 
  ?diseases_root meshv:treeNumber/^meshv:parentTreeNumber* ?tree.
  ?tree ^meshv:treeNumber ?id.
  ?id rdfs:label ?label .
  
  # ?diseases_rootのエントリにはparentが存在しないのでOPTIONALが必要
  OPTIONAL {
    ?tree meshv:parentTreeNumber/^meshv:treeNumber ?parent.
  }
  
  # 中間ノードの場合は?tree_childに値が存在し、leafノードの場合は?tree_childは存在しないのでOPTIONALが必要
  OPTIONAL {
    ?tree ^meshv:parentTreeNumber ?tree_child.
  }
  FILTER(lang(?label) = "en")
}
GROUP BY ?tree ?id ?parent ?label
```

# `return`
```javascript
({leaf, graph}) => {
  const idPrefix = "http://id.nlm.nih.gov/mesh/";
  let tree = [
    {
      id: "root",
      label: "root node",
      root: true
    }
  ];
  // Mesh親子関係
    graph.results.bindings.forEach(d => {
    tree.push({
      id: d.id.value.replace(idPrefix, ""),
      label: d.label.value,
      leaf: (d.tree_child == undefined ? true : false),
      parent: (d.parent == undefined ? "root" :  d.parent.value.replace(idPrefix, ""))
    });
  });
  // アノテーション関係
  leaf.results.bindings.map(d => {
    tree.push({
      id: d.child.value,
      label: d.child_label.value,
      leaf: true,
      parent: d.parent.value.replace(idPrefix, "")
    })
  })
  return tree;	
}
```