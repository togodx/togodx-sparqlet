# uniprot GO ìÒçÄä÷åW (éÁâÆ)

## Parameters

* `root` (type: go) (Req.)
  * default: GO_0005575
  * example: GO_0008150 (biological process), GO_0005575 (cellular component), GO_0003674 (molecular function), ... 

## Endpoint
https://integbio.jp/togosite/sparql

## `data`
```sparql
PREFIX up: <http://purl.uniprot.org/core/>
PREFIX taxon: <http://purl.uniprot.org/taxonomy/>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX obo: <http://purl.obolibrary.org/obo/>
PREFIX uniprot: <http://purl.uniprot.org/uniprot/>
SELECT DISTINCT ?parent ?child ?parent_label ?child_label ?leaf
FROM <http://rdf.integbio.jp/dataset/togosite/uniprot>
FROM <http://rdf.integbio.jp/dataset/togosite/go>
WHERE {
  VALUES ?root { obo:{{root}} }
  {
    ?child a up:Protein ;
           up:organism taxon:9606 ;
           up:proteome ?proteome ;
           up:classifiedWith ?parent .
    ?parent rdfs:subClassOf* ?root .
    ?child up:mnemonic ?child_label .
    BIND(1 AS ?leaf)
  } UNION {
    ?child rdfs:subClassOf* ?root ;
           rdfs:subClassOf ?parent .
    ?child rdfs:label ?child_label .
    ?protein up:classifiedWith/rdfs:subClassOf* ?child ;
             up:proteome ?proteome .
    FILTER(REGEX(STR(?child), "GO_"))
    FILTER(REGEX(STR(?parent), "GO_"))
    BIND(0 AS ?leaf)
  }
  FILTER(REGEX(STR(?proteome), "UP000005640"))
  ?parent rdfs:label ?parent_label .
}
```

## `allUniProt`
- UniProts without GO annotation 
```sparql
PREFIX up: <http://purl.uniprot.org/core/>
PREFIX taxon: <http://purl.uniprot.org/taxonomy/>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX obo: <http://purl.obolibrary.org/obo/>
PREFIX uniprot: <http://purl.uniprot.org/uniprot/>
SELECT DISTINCT ?uniprot ?label
FROM <http://rdf.integbio.jp/dataset/togosite/uniprot>
FROM <http://rdf.integbio.jp/dataset/togosite/go>
WHERE {
  ?uniprot a up:Protein ;
           up:organism taxon:9606 ;
           up:mnemonic ?label ;
           up:proteome ?proteome .
  FILTER(REGEX(STR(?proteome), "UP000005640"))
}
```

## `return`
```javascript
({root, data, allUniProt}) => {
  const idPrefix = "http://purl.uniprot.org/uniprot/";
  const categoryPrefix = "http://purl.obolibrary.org/obo/";
  
  let withGo = {};
  let tree = [{
    id: root,
    root: true
  },{
    id: "wo_" + root,
    label: "without annotation",
    parent: root
  }];
  
  data.results.bindings.map(d => {
    withGo[d.child.value] = true;
    tree.push({
      id: d.child.value.replace(categoryPrefix, "").replace(idPrefix, ""),
      label: d.child_label.value,
      leaf: Boolean(Number(d.leaf.value)),
      parent: d.parent.value.replace(categoryPrefix, "")
    })
    if (d.parent.value.replace(categoryPrefix, "") == root && !tree[0].label) tree[0].label = d.parent_label.value;
  })

  allUniProt.results.bindings.map(d => {
    if (!withGo[d.uniprot.value]) {
      tree.push({
        id: d.uniprot.value.replace(idPrefix, ""),
        label: d.label.value,
        leaf: true,
        parent: "wo_" + root,
      });
    }
  })
  
  return tree;	
}
```