# ChEMBL で薬を薬効（WHO ATC Code）で分類 (mode, hasChild対応, Number対応）（建石、山本） Server対応中未完

* 入力：
  * ATCコードのカテゴリ。（デフォルトは全部：この場合、パラメータは空白)
* 出力：
  * 入力したATCコードのカテゴリのサブカテゴリに含まれるChEMBL Molecule ID数（サブカテゴリ単位で集計）
  * 一つの薬に複数のATCコードがついていることがありうる 

## Description

- Data sources
    - ATC: https://bioportal.bioontology.org/ontologies/ATC
    - ChEMBL-RDF 28.0: http://ftp.ebi.ac.uk/pub/databases/chembl/ChEMBL-RDF/
- Query
    -  Input
        - ChEMBL ID
    - Output
        - ATC category ID
        - ATC category label

## Endpoint

https://integbio.jp/togosite/sparql

## `leaf`
- 化合物とATCコードの対応
- ソートしていない

```sparql
PREFIX cco: <http://rdf.ebi.ac.uk/terms/chembl#> 
PREFIX molecule: <http://rdf.ebi.ac.uk/resource/chembl/molecule/>

SELECT DISTINCT ?atc ?molecule
FROM <http://rdf.integbio.jp/dataset/togosite/chembl>  

WHERE {
  # test
      VALUES ?molecule {  molecule:CHEMBL17860  molecule:CHEMBL231779  molecule:CHEMBL231813  
                          molecule:CHEMBL251634  molecule:CHEMBL292707  molecule:CHEMBL312862  
                          molecule:CHEMBL43184  molecule:CHEMBL566315  molecule:CHEMBL600  molecule:CHEMBL63323  } 
  ?molecule cco:atcClassification ?atc .
        
}
```


## `atcGraph` 
-  

```javascript
({leaf})=>{
  let objects={}
  var a=leaf.results.bindings.map(d=>{
    let atc=d.atc.value;
    if (atc.length==7) {
      let child=atc;
      atc=atc.substr(0,5)
      objects[child]=atc 
    }
    if (atc.length==5) {
      let child=atc;
      atc=atc.substr(0,4)
      objects[child]=atc 
    }
    if (atc.length==4) {
      let child=atc;
      atc=atc.substr(0,3)
      objects[child]=atc 
    }
    if (atc.length==3) {
      let child=atc;
      atc=atc.substr(0,1)
      objects[child]=atc 
    }
  });	
  return(objects)
}
```


