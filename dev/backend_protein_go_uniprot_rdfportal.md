# UniProt GO classification for RDF portal (dev)(守屋)

- backend SPARQLet

## Parameters

* `root` (type: GO) (Req.)
  * default: GO_0005575
  * example: GO_0008150 (biological process), GO_0005575 (cellular component), GO_0003674 (molecular function)

## Endpoint
 https://integbio.jp/rdf/sib/sparql

## `leaf`
- GO と UniProt のアノテーション関係
```sparql
DEFINE sql:select-option "order"
PREFIX up: <http://purl.uniprot.org/core/>
PREFIX taxon: <http://purl.uniprot.org/taxonomy/>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX obo: <http://purl.obolibrary.org/obo/>
PREFIX xsd: <http://www.w3.org/2001/XMLSchema#>
SELECT DISTINCT ?parent ?child ?child_label
FROM <http://sparql.uniprot.org/uniprot>
FROM <http://sparql.uniprot.org/go> # GO は展開されているので、１ステップですべての祖先が取れる（が、元のGOの親子関係は取りにくい）
WHERE {
  GRAPH <http://sparql.uniprot.org/uniprot> {
    ?child up:organism taxon:9606 ;
           up:mnemonic ?child_label ;
           up:proteome ?proteome .
    FILTER (REGEX (STR ( ?proteome), "UP000005640"))
    ?child up:classifiedWith ?parent .
  }
  GRAPH <http://sparql.uniprot.org/go> { 
    ?parent rdfs:subClassOf obo:{{root}} . 
  }
}
```

## `allLeaf`
- 全 UniProt (without annotation 用)
```sparql
DEFINE sql:select-option "order"
PREFIX up: <http://purl.uniprot.org/core/>
PREFIX taxon: <http://purl.uniprot.org/taxonomy/>
SELECT DISTINCT ?leaf ?leaf_label
FROM <http://sparql.uniprot.org/uniprot>
WHERE {
  ?leaf up:organism taxon:9606 ;
        up:mnemonic ?leaf_label ;
        up:proteome ?proteome .
  FILTER (REGEX( STR( ?proteome), "UP000005640"))
}
```

## Endpoint
https://integbio.jp/rdf/bioportal/sparql
 
## `graph`
- GO の親子関係
  - 展開されていない endpoint で取得
  - human に無い GO も含まれるが、DX server では問題ない
```sparql
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX obo: <http://purl.obolibrary.org/obo/>
SELECT DISTINCT ?parent ?child ?parent_label ?child_label
FROM <http://rdf.integbio.jp/dataset/bioportal/go>
WHERE {
  ?child rdfs:subClassOf* obo:{{root}} ;
         rdfs:subClassOf ?parent .
  ?child rdfs:label ?child_label .
  ?parent rdfs:label ?parent_label .
}
```

## `return`
```javascript
({root, leaf, graph, allLeaf}) => {
  const idPrefix = "http://purl.uniprot.org/uniprot/";
  const categoryPrefix = "http://purl.obolibrary.org/obo/";
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
  // 親子関係
  graph.results.bindings.map(d => {
    tree.push({
      id: d.child.value.replace(categoryPrefix, ""),
      label: d.child_label.value,
      parent: d.parent.value.replace(categoryPrefix, "")
    })
    if (d.parent.value.replace(categoryPrefix, "") == root && !tree[0].label) tree[0].label = d.parent_label.value; // root の label 挿入
  })
  // アノテーション関係
  leaf.results.bindings.map(d => {
    withAnnotation[d.child.value] = true;
    tree.push({
      id: d.child.value.replace(idPrefix, ""),
      label: d.child_label.value,
      leaf: true,
      parent: d.parent.value.replace(categoryPrefix, "")
    })
  })
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