# Diseases in MeSH （三橋）

## Description

- Data sources
    -  [Medical Subject Headings (MeSH)](https://www.nlm.nih.gov/mesh/meshhome.html) 
- Query
    - Input
        - MeSH Descriptor
    - Output
        -  [Diseases ([C])](https://meshb.nlm.nih.gov/treeView) and its subcategories of MeSH

## Endpoint

https://integbio.jp/togosite/sparql

## testパターン
- この例は、[0_readme](https://togodx.integbio.jp/sparqlist_dev/0_readme)の「分類系：ルートノードが無い場合や、階層が無い場合」と近く、「分類系：ルートノードが無いが、階層がある場合」。
- parentがrootの例 ([Infections: D007239](https://meshb.nlm.nih.gov/record/ui?ui=D007239))
  - returnのJSONを「"id": "D007239"」で検索する
 ```
  {
    "id": "D007239",
    "label": "Infections",
    "leaf": false,
    "parent": "root"
  },
 ```
- parentが複数(DAG)かつleafの例 ([Aneurysm, Infected: D000785](https://meshb.nlm.nih.gov/record/ui?ui=D000785))
   -  returnのJSONを「"id": "D000785"」で検索する
```
  {
    "id": "D000785",
    "label": "Aneurysm, Infected",
    "leaf": true,
    "parent": "D007239"
  },
  {
    "id": "D000785",
    "label": "Aneurysm, Infected",
    "leaf": true,
    "parent": "D000783"
  },
```

## `data`

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

## `return`

```javascript
({data}) => {
  const idPrefix = "http://id.nlm.nih.gov/mesh/";
  let tree = [
    {
      id: "root",
      label: "root node",
      root: true
    }
  ];
  data.results.bindings.forEach(d => {
    tree.push({
      id: d.id.value.replace(idPrefix, ""),
      label: d.label.value,
      leaf: (d.tree_child == undefined ? true : false),
      parent: (d.parent == undefined ? "root" :  d.parent.value.replace(idPrefix, ""))
    });
  });
  return tree;
};
```