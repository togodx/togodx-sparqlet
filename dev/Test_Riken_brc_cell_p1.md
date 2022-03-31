# 理研BRCの細胞の情報　細胞の種類（高月）

## Description

- Data sources
     - RIKEN BRC CELL BANK: [https://cell.brc.riken.jp/en/](https://cell.brc.riken.jp/en/)
- Input/Output
     -  Input
        - Cell No.
    - Output
        - Category Type
- Supplementary information
     - ×××
     - RIKEN CELL BANKから提供されている細胞のリソース情報をカテゴリー別で分類


## Endpoint

https://knowledge.brc.riken.jp/sparql

## `data`

```sparql

PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX brs: <http://metadb.riken.jp/db/bioresource_schema/brs_>
PREFIX sio: <http://semanticscience.org/resource/SIO_>
PREFIX brso: <http://purl.org.jp/brso/>
PREFIX cell: <http://metadb.riken.jp/db/rikenbrc_cell/>

SELECT DISTINCT ?brcID ?cell_name ?category_label
WHERE {
   GRAPH <http://metadb.riken.jp/db/xsearch_cell> { 
     ?brcID rdfs:label ?cell_name.
     ?brcID cell:cell_0000057 ?categoryID.
     ?categoryID rdfs:label ?category_label.
     FILTER(lang(?category_label) = "en").
     }
  }
ORDER BY ?brcID
Limit 10

```
## `return`

```javascript
({data}) => {
  const idPrefix = "http://metadb.riken.jp/db/rikenbrc_mouse/";
 
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
      id: d.brcID.value.replace(idPrefix, ""),
      label: d.cell_name.value,
      leaf: true,
      parent: d.category_label.value
      })
  // root との親子関係を追加
    if (!edge[d.category_label.value]) {
      edge[d.category_label.value] = true;
      tree.push({   
        id: d.category_label.value,
        label: d.category_label.value,
        leaf: false,
        parent: "root"
      })
    }
  });
  
  return tree;
};
```


## MEMO
-Author
 - Takatsuki