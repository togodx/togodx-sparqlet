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

## `tf`

```sparql
PREFIX obo: <http://purl.obolibrary.org/obo/>
PREFIX ensembl: <http://identifiers.org/ensembl/>
PREFIX taxid: <http://identifiers.org/taxonomy/>
PREFIX faldo: <http://biohackathon.org/resource/faldo#>
PREFIX dc: <http://purl.org/dc/elements/1.1/>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>

SELECT DISTINCT ?parent
WHERE {
  GRAPH <http://rdf.integbio.jp/dataset/togosite/chip_atlas> {
    ?parent obo:RO_0002428 [] .
  }
} LIMIT 5
```

## `tfArray`
```javascript
({tf}) => {
  return tf.results.bindings.map(d => d.parent.value.replace("http://identifiers.org/ensembl/", ""));
}
```

## `return`
```javascript
//({main, noTf}) => {
async ({tf})=>{
  let tfArray = tf.results.bindings.map(d => d.parent.value.replace("http://identifiers.org/ensembl/", ""));
  let url = "backend_gene_transcription_factors_chip_atlas"; // parent SPARQLet relative path
  let options = {
    method: 'POST',
    headers: {
      'Accept': 'application/json',
      'Content-Type': 'application/x-www-form-urlencoded'
    }
  }
  return tfArray.map(
    (d) => {
      let obj = {};
      obj.parent = d;
      options.body = 'tfId=' + d;
      //let child_array = await fetch(url, options).then(res=>res.json());
      //obj.child: child_array;
      obj.option = options;
      return obj;
    }
  );
}
```
