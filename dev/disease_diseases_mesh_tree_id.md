# Diseases in MeSH （三橋,守屋）

* MeSH の D番号を node ID として使うと、情報が減って DAG がループしてしまうので、Tree番号をIDとする
  * 中間ノードへのIDのマッピングが効かないのでお蔵入り

## Description

- Data sources
    -  [Medical Subject Headings (MeSH)](https://www.nlm.nih.gov/mesh/meshhome.html) 

## Endpoint

{{SPARQLIST_TOGODX_SPARQL}}

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
PREFIX meshv: <http://id.nlm.nih.gov/mesh/vocab#>

SELECT DISTINCT ?mesh_id ?mesh_label ?parent_mesh_id ?node ?parent (SAMPLE(?child) AS ?tree_child)
FROM <http://rdf.integbio.jp/dataset/togosite/mesh>
WHERE {
  # MeSH Treeの Diseases[C] 以下を取得
  # See https://meshb.nlm.nih.gov/treeView
  FILTER (contains(str(?node), "C"))
  OPTIONAL {
    ?node meshv:parentTreeNumber ?parent  .
    ?parent_mesh_id meshv:treeNumber ?parent .
  }

  ?mesh_id meshv:treeNumber ?node ;
    rdfs:label ?mesh_label .
  FILTER(lang(?mesh_label) = "en")
  
  OPTIONAL {
    ?child meshv:parentTreeNumber ?node.
  }
}
GROUP BY ?mesh_id ?mesh_label ?parent_mesh_id ?node ?parent
```

## `return`

```javascript
({data}) => {
  const idPrefix = "http://id.nlm.nih.gov/mesh/";
  let tree = [
    {
      id: "root",
      label: "Diseases",
      root: true
    }
  ];
  data.results.bindings.forEach(d => {
    tree.push({
      id: (d.tree_child == undefined ? d.mesh_id.value.replace(idPrefix, "") : d.node.value.replace(idPrefix, "")), 
      label: d.mesh_label.value,
      leaf: (d.tree_child == undefined ? true : false),
      parent: (d.parent == undefined ? "root" :  d.parent.value.replace(idPrefix, "")),
      ontology_leaf: d.mesh_id.value.replace(idPrefix, "")
    });
  });
  return tree;
};
```