# Genes on chromosomes (片山、池田、守屋)

## Description

- Data sources
    - Ensembl human release 102: [http://nov2020.archive.ensembl.org/Homo_sapiens/Info/Index](http://nov2020.archive.ensembl.org/Homo_sapiens/Info/Index)
- Input/Output
    -  Input
        - Ensembl gene ID
    - Output
        - Chromosome number
- Supplementary information
    - The chromosome on which each human gene is located.
    - ヒトの各遺伝子が位置している染色体の番号を示します。

## Endpoint

https://integbio.jp/togosite/sparql

## `data`

```sparql
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX dc: <http://purl.org/dc/elements/1.1/>
PREFIX obo: <http://purl.obolibrary.org/obo/>
PREFIX ensg: <http://rdf.ebi.ac.uk/resource/ensembl/>
PREFIX enso: <http://rdf.ebi.ac.uk/terms/ensembl/>
PREFIX faldo: <http://biohackathon.org/resource/faldo#>
PREFIX taxonomy: <http://identifiers.org/taxonomy/>
SELECT DISTINCT ?parent ?child ?child_label
FROM <http://rdf.integbio.jp/dataset/togosite/ensembl>
WHERE {
  ?child faldo:location ?ensg_location ;
         rdfs:label ?child_label ;
         obo:RO_0002162 taxonomy:9606  .
  ?transcript obo:SO_transcribed_from ?child .
  BIND (strbefore(strafter(str(?ensg_location), "GRCh38/"), ":") AS ?parent)
  VALUES ?parent {"1" "2" "3" "4" "5" "6" "7" "8" "9" "10" "11" "12" "13" "14" "15" "16" "17" "18" "19" "20" "21" "22" "X" "Y" "MT"}
}
```

## `return`

```javascript
({data}) => {
  const idPrefix = "http://rdf.ebi.ac.uk/resource/ensembl/";
  
  let tree = [
    {
      id: "root",
      label: "root node",
      root: true
    }
  ];

  let edge = {};
  data.results.bindings.map(d => {
    tree.push({
      id: d.child.value.replace(idPrefix, ""),
      label: d.child_label.value,
      leaf: true,
      parent: d.parent.value
    })
    // root との親子関係を追加
    if (!edge[d.parent.value]) {
      edge[d.parent.value] = true;
      tree.push({     
        id: d.parent.value,
        label: "chr" + d.parent.value,
        leaf: false,
        parent: "root"
      })
    }
  });
  
  return tree;
};
```