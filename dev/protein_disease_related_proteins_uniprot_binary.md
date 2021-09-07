# uniprot keywords 二項関係（守屋）

* UniProt keyword の内訳
  * ?uniprot up:classifiedWith [keyword ID]
  * [keyword ID] rdfs:subClassOF+ [10 keyword types]
    * 9990: Technical term (1階層 (階層なし))
    * 9991: PTM (最大3階層)
    * 9992: Molecular function (最大5階層)
    * 9993: Ligand (最大5階層)
    * 9994: Domain (最大2階層)
    * 9995: Disease (最大3階層)
    * (9996: Developmental stage (最大3階層) human 無し)
    * 9997: Coding sequence diversity (1階層 (階層なし))
    * 9998: Cellular component (最大4階層)
    * 9999: Biological process (最大8階層)

## Parameters

* `root` (type: UniProt keyword ID) (Req.)
  * default: 9995

## Endpoint
https://integbio.jp/togosite/sparql

## `data`
- Attributeの階層関係、Attributeと要素（UniProt）の関係を二項関係で取得
```sparql
PREFIX up: <http://purl.uniprot.org/core/>
PREFIX taxon: <http://purl.uniprot.org/taxonomy/>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX skos: <http://www.w3.org/2004/02/skos/core#>
PREFIX uniprot: <http://purl.uniprot.org/uniprot/>
PREFIX keywords: <http://purl.uniprot.org/keywords/>
SELECT DISTINCT ?parent ?child ?parent_label ?child_label ?leaf
FROM <http://rdf.integbio.jp/dataset/togosite/uniprot>
FROM <http://rdf.integbio.jp/dataset/togosite/uniprot/keywords>
WHERE {
  VALUES ?root { keywords:{{root}} }
  {
    ?child a up:Protein ;
           up:organism taxon:9606 ;
           up:proteome ?proteome ;
           up:classifiedWith ?parent .
    FILTER(REGEX(STR(?proteome), "UP000005640"))
    ?parent a up:Concept ;
            rdfs:subClassOf* ?root .
    ?child up:mnemonic ?child_label .
    BIND(1 AS ?leaf)
  } UNION {
    ?child rdfs:subClassOf* ?root ;
           a up:Concept ;
           rdfs:subClassOf ?parent .
    ?parent rdfs:subClassOf* ?root . # keywords は複数のカテゴリにぶら下がることがあるので親のrootもチェック
    ?child skos:prefLabel ?child_label .
    BIND(0 AS ?leaf)
  }
  ?parent skos:prefLabel ?parent_label .
}
```

## `return`
- 整形
```javascript
({root, data})=>{
  const idPrefix = "http://purl.uniprot.org/uniprot/";
  const categoryPrefix = "http://purl.uniprot.org/keywords/";

  let tree = [{
    id: root,
    root: true
  }];
  
  data.results.bindings.map(d => {
    tree.push({
      id: d.child.value.replace(categoryPrefix, "").replace(idPrefix, ""),
      label: d.child_label.value,
      leaf: Boolean(Number(d.leaf.value)),
      parent: d.parent.value.replace(categoryPrefix, "")
    })
    if (d.parent.value.replace(categoryPrefix, "") == root && !tree[0].label) tree[0].label = d.parent_label.value;
  });
  
  return tree;
}
```