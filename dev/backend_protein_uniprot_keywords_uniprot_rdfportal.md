# UniProt keywords classification for RDF portal（守屋）

- backend SPARQLet

- UniProt keyword の内訳
  - ?uniprot up:classifiedWith [keyword ID]
  - [keyword ID] rdfs:subClassOF+ [10 keyword types]
    - 9990: Technical term (1階層 (階層なし))
    - 9991: PTM (最大3階層)
    - 9992: Molecular function (最大5階層)
    - 9993: Ligand (最大5階層)
    - 9994: Domain (最大2階層)
    - 9995: Disease (最大3階層)
    - (9996: Developmental stage (最大3階層) human 無し)
    - 9997: Coding sequence diversity (1階層 (階層なし))
    - 9998: Cellular component (最大4階層)
    - 9999: Biological process (最大8階層)

## Parameters

* `root` (type: UniProt keyword ID) (Req.)
  * default: 9995

## Endpoint
https://rdfportal.org/sib/sparql

## `graph`
- keyword ID と UniProt ID のアノテーション関係と、keyword ID の親子関係を同時取得
```sparql
DEFINE sql:select-option "order"
PREFIX up: <http://purl.uniprot.org/core/>
PREFIX taxon: <http://purl.uniprot.org/taxonomy/>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX skos: <http://www.w3.org/2004/02/skos/core#>
PREFIX uniprot: <http://purl.uniprot.org/uniprot/>
PREFIX keywords: <http://purl.uniprot.org/keywords/>
PREFIX chu: <http://purl.uniprot.org/proteomes/UP000005640#Unplaced>
PREFIX ch: <http://purl.uniprot.org/proteomes/UP000005640#Chromosome%20>
SELECT DISTINCT ?parent ?child ?parent_label ?child_label ?leaf
FROM <http://sparql.uniprot.org/uniprot>
FROM <http://sparql.uniprot.org/keywords>
WHERE {
  VALUES ?proteome { ch:1 ch:2 ch:3 ch:4 ch:5 ch:6 ch:7 ch:8 ch:9 ch:10 
                     ch:11 ch:12 ch:13 ch:14 ch:15 ch:16 ch:17 ch:18 ch:19
                     ch:20 ch:21 ch:22 ch:X ch:Y ch:MT chu: }
  {
    GRAPH <http://sparql.uniprot.org/uniprot> {
      ?child up:mnemonic ?child_label ;
             up:proteome ?proteome .
      ?child up:classifiedWith ?parent .
    }
    GRAPH <http://sparql.uniprot.org/keywords> {
      ?parent a up:Concept ;
              rdfs:subClassOf* keywords:{{root}} .
    }
    BIND(1 AS ?leaf)
  } UNION {
    GRAPH <http://sparql.uniprot.org/keywords> {
      ?child rdfs:subClassOf* keywords:{{root}} ;
             a up:Concept ;
             rdfs:subClassOf ?parent .
      ?parent rdfs:subClassOf* keywords:{{root}} . # keywords は複数のカテゴリにぶら下がることがあるので親のrootもチェック
      ?child skos:prefLabel ?child_label .
    }
    BIND(0 AS ?leaf)
  }
  GRAPH <http://sparql.uniprot.org/keywords> {
    ?parent skos:prefLabel ?parent_label .
  }
}
```

## `allLeaf`
- 全 UniProt (without annotation 用)
```sparql
PREFIX up: <http://purl.uniprot.org/core/>
PREFIX taxon: <http://purl.uniprot.org/taxonomy/>
PREFIX chu: <http://purl.uniprot.org/proteomes/UP000005640#Unplaced>
PREFIX ch: <http://purl.uniprot.org/proteomes/UP000005640#Chromosome%20>
SELECT DISTINCT ?leaf ?leaf_label
FROM <http://sparql.uniprot.org/uniprot>
WHERE {
  VALUES ?proteome { ch:1 ch:2 ch:3 ch:4 ch:5 ch:6 ch:7 ch:8 ch:9 ch:10 
                     ch:11 ch:12 ch:13 ch:14 ch:15 ch:16 ch:17 ch:18 ch:19
                     ch:20 ch:21 ch:22 ch:X ch:Y ch:MT chu: }
  ?leaf a up:Protein ;
        up:mnemonic ?leaf_label ;
        up:proteome ?proteome .
}
```

## `return`
```javascript
({root, graph, allLeaf})=>{
  const idPrefix = "http://purl.uniprot.org/uniprot/";
  const categoryPrefix = "http://purl.uniprot.org/keywords/";
  const withoutId = "unclassified";
  
  let tree = [
    {
      id: root,
      root: true
    },{
      id: withoutId,
      label: "Unclassified",
      parent: root
    }
  ];

  let withAnnotation = {};
  // 親子関係とアノテーション関係
  graph.results.bindings.map(d => {
    withAnnotation[d.child.value] = true;
    tree.push({
      id: d.child.value.replace(categoryPrefix, "").replace(idPrefix, ""),
      label: d.child_label.value,
      leaf: Boolean(Number(d.leaf.value)),
      parent: d.parent.value.replace(categoryPrefix, "")
    })
    if (d.parent.value.replace(categoryPrefix, "") == root && !tree[0].label) tree[0].label = d.parent_label.value; // root の label 挿入
  });
  // アノテーション無し要素
  allLeaf.results.bindings.map(d => {
    if (!withAnnotation[d.leaf.value]) {
      tree.push({
        id: d.leaf.value.replace(idPrefix, ""),
        label: d.leaf_label.value,
        leaf: true,
        parent: withoutId
      });
    }
  })
  
  return tree;
}
```