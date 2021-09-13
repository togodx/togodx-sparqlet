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