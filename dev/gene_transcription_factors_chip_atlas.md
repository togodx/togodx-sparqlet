# Target genes of TFs in ChIP-Atlas （大田・池田・小野・千葉）

## Description

- Data sources
    - ChIP-Atlas: [http://dbarchive.biosciencedbc.jp/kyushu-u/hg38/target/](http://dbarchive.biosciencedbc.jp/kyushu-u/hg38/target/)
    - The genes in `<Transcription Factor>.10.tsv` were defined to be target genes of the `<Transcription Factor>`.

- Query
    - Input
        - Ensembl gene ID of target genes
    - Output
        - Ensembl gene ID of TFs

## Endpoint

https://integbio.jp/togosite/sparql

## `main`

```sparql
PREFIX obo: <http://purl.obolibrary.org/obo/>
PREFIX ensembl: <http://identifiers.org/ensembl/>
PREFIX taxid: <http://identifiers.org/taxonomy/>
PREFIX faldo: <http://biohackathon.org/resource/faldo#>
PREFIX dc: <http://purl.org/dc/elements/1.1/>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>

SELECT DISTINCT ?parent ?child
WHERE {
  VALUES ?parent { ensembl:ENSG00000004487 ensembl:ENSG00000005339 }
  GRAPH <http://rdf.integbio.jp/dataset/togosite/chip_atlas> {
    ?parent obo:RO_0002428 ?child .
  }
}
```


## `return`
```javascript
//({main, noTf}) => {
({main}) => {
  let tree = [{
    id: "root",
    label: "root node",
    root: true
  }];
  let chk = {};
  //let data = main.results.bindings.concat(noTf.results.bindings);
  let data = main.results.bindings;
  data.map(d => {
    if (!chk[d.parent.value]) {
      chk[d.parent.value] = true;
      tree.push({     
        id: d.parent.value,
        label: d.parent.value,
        leaf: false,
        parent: "root"
      })
    }
    tree.push({
      id: d.child.value,
      label: d.child.value,
      leaf: true,
      parent: d.parent.value
    })
  });
  
  return tree;
};
```
