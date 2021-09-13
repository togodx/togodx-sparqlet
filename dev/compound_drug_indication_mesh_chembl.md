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

```

## `targetMesh`
- mesh D番号 と目的 tree 階層の対応表
  - Top レベルだけ例外処理
```sparql
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX meshv: <http://id.nlm.nih.gov/mesh/vocab#>
PREFIX tree: <http://id.nlm.nih.gov/mesh/>
SELECT DISTINCT ?mesh ?tree ?label (SAMPLE(?child_tree_ori) AS ?child_tree)
#FROM <http://integbio.jp/rdf/mirror/ontology/mesh>
FROM <http://rdf.integbio.jp/dataset/togosite/mesh>
WHERE {
  ?tree a meshv:TreeNumber .
  MINUS { 
    ?tree meshv:parentTreeNumber ?parent .
  }
  FILTER (CONTAINS(STR(?tree),"mesh/C"))
   ?tree ^meshv:treeNumber/rdfs:label ?label .
   ?mesh meshv:treeNumber/meshv:parentTreeNumber* ?tree .
   OPTIONAL {
     ?child_tree_ori meshv:parentTreeNumber ?tree .
   }
}
```

## `data`
- ChEMBL molecule - mesh D番号の対応リスト
```sparql
PREFIX cco: <http://rdf.ebi.ac.uk/terms/chembl#> 
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
SELECT DISTINCT?child ?child_label ?parent 
FROM <http://rdf.integbio.jp/dataset/togosite/chembl>
WHERE {
  ?molecule cco:chemblId ?child ;
            rdfs:label ?child_label ;
            cco:hasDrugIndication [
    a cco:DrugIndication ;
    cco:hasMesh ?parent
  ] .
}
```
## `return`
```javascript
({targetMesh, data})=>{
 let meshExist = {};
  for (let d of targetMesh.results.bindings) {
    meshExist[d.mesh.value.replace("http://id.nlm.nih.gov/mesh/", "")] = true;
  }
  let mesh2id = {};
  let id2label = {};
  let id2child_tree = {};
  for (let d of targetMesh.results.bindings) {
    let mesh = d.mesh.value.replace("http://id.nlm.nih.gov/mesh/", "");
    let id = d.tree.value.replace("http://id.nlm.nih.gov/mesh/", ""); // tree number
    if (!mesh2id[mesh]) mesh2id[mesh] = [id];
    mesh2id[mesh].push(id);
    if (!id2label[id]) id2label[id] = d.label.value;
    id2child_tree[id] = Boolean(d.child_tree);
  }
  let id2chembl = {};
  let filteredList = [];
  for (let d of data.results.bindings) {
    let parent = d.parent.value.replace("http://identifiers.org/mesh/", "");
    let child = d.child.value;
    if (meshExist[parent] && (!chemblFilterExist || chemblFilterExist[child])) {
      for (let id of mesh2id[parent]) {
        if (!id2chembl[id]) id2chembl[id] = [];
        id2chembl[id].push(child);　// tree number と chembl 対応
        filteredList.push({
          id: child,
          attribute: {
            categoryId: id,
            uri: "http://id.nlm.nih.gov/mesh/" + id,
            label: id2label[id]
          }
        })
      }
    }
  }
  let id2count = {};
  for (let id of Object.keys(id2chembl)) {
    id2count[id] = Array.from(new Set(id2chembl[id])).length;  // chembl を unique してカウント
  }
  if (mode == "objectList") return filteredList;
  if (mode == "idList") return Array.from(new Set(filteredList.map(d=>d.id))); // chembl を unique
  
  return Object.keys(id2count).sort((a,b)=>{
    if (id2count[a] < id2count[b]) return 1;
    if (id2count[a] > id2count[b]) return -1;
    return 0;
  }).map(id=>{
    return {
      categoryId: id,
      label: id2label[id],
      count: id2count[id],
      hasChild: id2child_tree[id]
    }
  });                        
}
```