# UniProt with reviewed flag classification（守屋）

## Endpoint
https://integbio.jp/togosite/sparql

## `graph`
```sparql
PREFIX up: <http://purl.uniprot.org/core/>
SELECT DISTINCT ?child ?child_label
FROM <http://rdf.integbio.jp/dataset/togosite/uniprot>
WHERE {
  ?child a up:Protein ;
         up:mnemonic ?child_label ;
         up:proteome ?proteome ;
         up:reviewed 1 .
  FILTER(REGEX(STR(?proteome), "UP000005640"))
}
```

## `allLeaf`
```sparql
PREFIX up: <http://purl.uniprot.org/core/>
SELECT DISTINCT ?leaf ?leaf_label
FROM <http://rdf.integbio.jp/dataset/togosite/uniprot>
WHERE {
  ?leaf a up:Protein ;
        up:mnemonic ?leaf_label ;
        up:proteome ?proteome .
  FILTER(REGEX(STR(?proteome), "UP000005640"))
}
```

## `return`
```javascript
({graph, allLeaf})=>{
  const idPrefix = "http://purl.uniprot.org/uniprot/";
  const withoutId = "unclassified";
  
  let tree = [
    {
      id: "root",
      root: true
    },{
      id: "reviewed",
      label: "Swiss-Prot",
      parent: "root"
    },{
      id: withoutId,
      label: "TrEMBL",
      parent: "root"
    }
  ];

  let withAnnotation = {};
  // 親子関係とアノテーション関係
  graph.results.bindings.map(d => {
    withAnnotation[d.child.value] = true;
    tree.push({
      id: d.child.value.replace(idPrefix, ""),
      label: d.child_label.value,
      leaf: true,
      parent: "reviewed"
    })
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