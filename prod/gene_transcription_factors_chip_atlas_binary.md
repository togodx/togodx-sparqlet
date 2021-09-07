# ChIP-Atlas の転写因子を GO BP で分類 二項関係

## Parameters

* 古い？ TF を biological process で分類してたころのやつ
* `root` (type: go) (Req.)
  * default: GO_0008150
  * example: GO_0008150 (biological process)

## Endpoint
https://integbio.jp/togosite/sparql

## `targetTf`
```sparql
PREFIX obo: <http://purl.obolibrary.org/obo/>
PREFIX ensg: <http://identifiers.org/ensembl/>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
SELECT DISTINCT ?tf_ensg
FROM <http://rdf.integbio.jp/dataset/togosite/chip_atlas>
WHERE {
  ?tf_ensg obo:RO_0002428 ?target .
}
```
## `targetTfArray`
```javascript
({targetTf}) => {
  return targetTf.results.bindings.map(d => d.tf_ensg.value.replace("http://identifiers.org/ensembl/", ""));
}
```

## `data`
```sparql
PREFIX up: <http://purl.uniprot.org/core/>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX obo: <http://purl.obolibrary.org/obo/>
PREFIX ensg: <http://rdf.ebi.ac.uk/resource/ensembl/>
PREFIX taxon: <http://purl.uniprot.org/taxonomy/>
SELECT DISTINCT ?parent ?child ?parent_label ?child_label ?leaf
FROM <http://rdf.integbio.jp/dataset/togosite/uniprot>
FROM <http://rdf.integbio.jp/dataset/togosite/go>
WHERE {
  VALUES ?root { obo:{{root}} }
  {
    ?parent ^up:classifiedWith [
         up:organism taxon:9606 ;
         rdfs:seeAlso/up:transcribedFrom ?child 
       ] ;
       rdfs:subClassOf* ?root .
    BIND(1 AS ?leaf)
  } UNION {
    ?child rdfs:subClassOf* ?root ;
           rdfs:subClassOf ?parent .
    ?child rdfs:label ?child_label .
   FILTER(REGEX(STR(?child), "GO_"))
    BIND(0 AS ?leaf) 
  }
  ?parent rdfs:label ?parent_label .
}
```

## `ensgLabel`
```sparql
PREFIX ensg: <http://rdf.ebi.ac.uk/resource/ensembl/>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
SELECT DISTINCT ?ensg ?label
FROM <http://rdf.integbio.jp/dataset/togosite/ensembl>
WHERE {
  VALUES ?ensg{ {{#each targetTfArray}} ensg:{{this}} {{/each}} }
  ?ensg rdfs:label ?label .
}
```

## `return`
```javascript
({root, data, targetTf, ensgLabel}) => {
  const idPrefix = "http://identifiers.org/ensembl/";
  const idPrefix2 = "http://rdf.ebi.ac.uk/resource/ensembl/";
  const categoryPrefix = "http://purl.obolibrary.org/obo/";
  
  let geneLabel = {};
  ensgLabel.results.bindings.map(d => {
    geneLabel[d.ensg.value.replace(idPrefix2, "")] = d.label.value;
  })
                                 
  let withTfGene = {};
  targetTf.results.bindings.map(d => {
    withTfGene[d.tf_ensg.value.replace(idPrefix, "")] = true;
  })
                                
  let withGo = {};
  let tree = [{
    uri: categoryPrefix + root,
    id: root,
    root: true
  },{
    //uri: "without_annotation",
    id: "wo_" + root,
    label: "without annotation",
    parent: root
  }];
  
  data.results.bindings.map(d => {
    if (Number(d.leaf.value) == 0 || withTfGene[d.child.value.replace(idPrefix2, "")]) {
      withGo[d.child.value] = true;
      let child_label = "";
      if (d.child_label) child_label = d.child_label.value;
      else child_label = geneLabel[d.child.value.replace(idPrefix2, "")];
      tree.push({
        id: d.child.value.replace(categoryPrefix, "").replace(idPrefix2, ""),
        label: child_label,
        leaf: Boolean(Number(d.leaf.value)),
        parent: d.parent.value.replace(categoryPrefix, "")
      })
      if (d.parent.value.replace(categoryPrefix, "") == root && !tree[0].label) tree[0].label = d.parent_label.value;
    }
  })

  targetTf.results.bindings.map(d => {
    if (!withGo[d.tf_ensg.value]) {
      tree.push({
        id: d.tf_ensg.value.replace(idPrefix, ""),
        label: geneLabel[d.tf_ensg.value.replace(idPrefix, "")],
        leaf: true,
        parent: "wo_" + root
      })
    }
  })
  
  return tree;	
}
```