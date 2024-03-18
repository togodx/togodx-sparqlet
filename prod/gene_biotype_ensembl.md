# Ensembl gene type （池田）

## Description

- Data sources
    - Ensembl human release 111: [https://useast.ensembl.org/Homo_sapiens/Info/Index](https://useast.ensembl.org/Homo_sapiens/Info/Index)
- Output
    - Gene type
 - Supplementary information
 	- A gene or transcript classification such as "protein coding" and "lncRNA". Definition of gene biotypes is described [here](http://useast.ensembl.org/info/genome/genebuild/biotypes.html).
	- protein coding, lncRNA といった遺伝子/転写産物の分類です。遺伝子の分類の定義については [こちらをご覧ください](http://useast.ensembl.org/info/genome/genebuild/biotypes.html)。

## Endpoint

{{SPARQLIST_TOGODX_SPARQL}}

## `data`

```sparql
PREFIX obo: <http://purl.obolibrary.org/obo/>
PREFIX so: <http://purl.obolibrary.org/obo/so#>
PREFIX taxon: <http://identifiers.org/taxonomy/>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX faldo: <http://biohackathon.org/resource/faldo#>
PREFIX dcterms: <http://purl.org/dc/terms/>
PREFIX terms: <http://rdf.ebi.ac.uk/terms/ensembl/>

SELECT DISTINCT ?parent ?parent_label ?child ?child_label
FROM <http://rdf.integbio.jp/dataset/togosite/ensembl>
FROM <http://rdf.integbio.jp/dataset/togosite/so>
WHERE {
  ?enst so:transcribed_from ?ensg .
  ?ensg terms:has_biotype ?parent ;
        obo:RO_0002162 taxon:9606 ;
        faldo:location ?ensg_location ;
        dcterms:identifier ?child ;
        rdfs:label ?child_label ;
        so:part_of ?chr .
  ?parent rdfs:label ?parent_label .
  # FILTER(CONTAINS(STR(?parent), "terms/ensembl/"))
  BIND(STRBEFORE(STRAFTER(STR(?chr), "hco/"), "/GRCh38") AS ?chromosome)
  VALUES ?chromosome {
      "1" "2" "3" "4" "5" "6" "7" "8" "9" "10"
      "11" "12" "13" "14" "15" "16" "17" "18" "19" "20" "21" "22"
      "X" "Y" "MT"
  }
}
```

## `return`

```javascript
({data}) => {
  const idPrefixes = ["http://rdf.ebi.ac.uk/terms/ensembl/", "http://purl.obolibrary.org/obo/", "http://ensembl.org/glossary/"];
  const pattern = new RegExp(idPrefixes.join("|"), "g")
  
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
        id: d.parent.value.replace(pattern, ""),
        label: d.parent_label.value.replace(/_/g, " "),
        leaf: false,
        parent: "root"
      })
    }
    let label = d.child_label.value;
    if (label == "") {
      label = d.child.value;
    }
    tree.push({
      id: d.child.value,
      label: label,
      leaf: true,
      parent: d.parent.value.replace(pattern, "")
    })
  });
  
  return tree;
};
```