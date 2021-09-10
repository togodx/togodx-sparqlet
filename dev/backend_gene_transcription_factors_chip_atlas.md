# Target genes of TFs in ChIP-Atlas （大田・池田・小野・千葉）

Input a ENSG ID of a TF and output a list of ENSG IDs of downstream genes

## Description

- Data sources
    - ChIP-Atlas: [http://dbarchive.biosciencedbc.jp/kyushu-u/hg38/target/](http://dbarchive.biosciencedbc.jp/kyushu-u/hg38/target/)
    - The genes in `<Transcription Factor>.10.tsv` were defined to be target genes of the `<Transcription Factor>`.


## Parameters
* `tfId` (type: ensembl gene ID (TF))
  * example: ENSG00000275700,ENSG00000101544,ENSG00000048052

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

SELECT DISTINCT ?child
WHERE {
  VALUES ?parent { ensembl:{{tfId}} }
  GRAPH <http://rdf.integbio.jp/dataset/togosite/chip_atlas> {
    ?parent obo:RO_0002428 ?child .
  }
}
```

## `return`
```javascript
({main}) => {
  return main.results.bindings.map(d => d.child.value.replace("http://identifiers.org/ensembl/", ""));
};
```
