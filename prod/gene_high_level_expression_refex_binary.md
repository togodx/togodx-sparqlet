# RefEx organ specific expression based on ROKU flag 二項関係（守屋）

## Parameters

* `negatively` (低発現)
  * example: 1

## Endpoint
https://integbio.jp/togosite/sparql

## `data`

```sparql
PREFIX refexo: <http://purl.jp/bio/01/refexo#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX hop: <http://purl.org/net/orthordf/hOP/ontology#>
SELECT DISTINCT ?parent ?child ?parent_label ?child_label ?label
FROM <http://rdf.integbio.jp/dataset/togosite/refexo>
FROM <http://rdf.integbio.jp/dataset/togosite/refex_id_relation_human>
FROM <http://rdf.integbio.jp/dataset/togosite/refex_tissue_specific_genechip_human_GSE7307>
#FROM <http://rdf.integbio.jp/dataset/togosite/refex_tissue_specific_rnaseq_human_PRJEB2445>
FROM <http://rdf.integbio.jp/dataset/togosite/homo_sapiens_gene_info>
WHERE {
  ?child refexo:affyProbeset ?affy ;
         hop:fullName ?child_label .
{{#if negatively}}
  ?affy refexo:isNegativelySpecificTo ?parent .
{{else}}
  ?affy refexo:isPositivelySpecificTo ?parent .
{{/if}}
  ?parent rdfs:label ?parent_label .
  FILTER (LANG (?parent_label) = "en")
}
```

## `return`

```javascript
({data})=>{
  const idPrefix = "http://identifiers.org/ncbigene/";
  const categoryPrefix = "http://purl.jp/bio/01/refexo#";

  let tree = [{
    id: "root",
    label: "root node",
    root: true
  }];
  let chk = {};
  
  data.results.bindings.map(d => {
    if (!chk[d.parent.value]) {
      chk[d.parent.value] = true;
      tree.push({
        id: d.parent.value.replace(categoryPrefix, ""),
        label: d.parent_label.value,
        parent: "root"
      })
    }
    tree.push({
      id: d.child.value.replace(idPrefix, ""),
      label: d.child_label.value,
      leaf: true,
      parent: d.parent.value.replace(categoryPrefix, "")
    })
  })
  
  return tree;
}
```