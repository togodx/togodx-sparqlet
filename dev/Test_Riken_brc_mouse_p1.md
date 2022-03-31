# 理研BRCのマウス情報　作出方法別（高月）

## Description

- Data sources
     - RIKEN BRC Experimental Animal Division: [https://mus.brc.riken.jp/en/](https://mus.brc.riken.jp/en/)
- Input/Output
     -  Input
        - BRC No.
    - Output
        - Strain Type
- Supplementary information
     - ×××
     - RIKEN BRC実験動物開発室から提供されているマウスのリソース情報をタイプ別で分類


## Endpoint

https://knowledge.brc.riken.jp/sparql

## `data`

```sparql

PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX brs: <http://metadb.riken.jp/db/bioresource_schema/brs_>

SELECT DISTINCT ?brcID ?category_label ?mouse_label
WHERE {
   GRAPH <http://metadb.riken.jp/db/xsearch_animal> { 
    ?brcID    rdfs:label ?mouse_label;
              brs:strainTypeLink ?category.
    ?category rdfs:label ?category_label.
  }
} 
ORDER BY ?brcID

#LIMIT 10
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
      label: d.mouse_label.value,
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