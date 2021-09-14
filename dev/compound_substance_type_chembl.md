# ChEMBLをsubstancetypeで分類する（信定・鈴木・八塚） 作業中（多数問題）
現在sparqlの結果数を制限中

## Description

- Data sources
    - (More data sources description goes here..)
    - ChEMBL-RDF 28.0: http://ftp.ebi.ac.uk/pub/databases/chembl/ChEMBL-RDF/
- Query
    - (More query details go here..)
    -  Input
        - ChEMBL ID
    - Output
        - Substance type

# Parameters
* `offsetNumber`
  * default: 0, 5
  
## `offsetArray` 
- offsetNumber を配列に分割
```javascript
({offsetNumber}) => {
  offsetNumber = offsetNumber.replace(/,/g," ")
  if (offsetNumber.match(/[^\s]/)) return offsetNumber.split(/\s+/);
  return false;
}
```

## Endpoint

https://integbio.jp/togosite/sparql

## `data`

```sparql
PREFIX cco: <http://rdf.ebi.ac.uk/terms/chembl#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
SELECT DISTINCT?parent ?child ?child_label
FROM <http://rdf.integbio.jp/dataset/togosite/chembl>
WHERE 
{
  ?substance cco:substanceType ?parent ;
                       cco:chemblId  ?child ;
             rdfs:label ?child_label .
             }
OFFSET   { {{#each offsetArray}} {{this}} {{/each}}}
LIMIT 20
```
## `return`

```javascript
({data}) => {
  
  let tree = [
    {
      id: "root",
      label: "root node",
      root: true
    }
  ];

  let edge = {};
  data.results.bindings.map(d => {
    tree.push({
      id: d.child.value,
      label: d.child_label.value,
      leaf: true,
      parent: d.parent.value
    })
  // root との親子関係を追加
    if (!edge[d.parent.value]) {
      edge[d.parent.value] = true;
      tree.push({   
        id: d.parent.value,
        label: d.parent.value,
        leaf: false,
        parent: "root"
      })
    }
  });
  
  return tree;
};
```