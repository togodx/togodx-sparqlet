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

SELECT DISTINCT ?parent
WHERE {
  GRAPH <http://rdf.integbio.jp/dataset/togosite/chip_atlas> {
    ?parent obo:RO_0002428 [] .
  }
} LIMIT 5
```

## `geneLabels`
```sparql
PREFIX obo: <http://purl.obolibrary.org/obo/>
PREFIX ensembl: <http://identifiers.org/ensembl/>
PREFIX taxid: <http://identifiers.org/taxonomy/>
PREFIX faldo: <http://biohackathon.org/resource/faldo#>
PREFIX dc: <http://purl.org/dc/elements/1.1/>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>

SELECT DISTINCT ?ensg_id ?ensg_label
WHERE {
  GRAPH <http://rdf.integbio.jp/dataset/togosite/ensembl> {
    ?enst obo:SO_transcribed_from ?ensg .
    ?ensg obo:RO_0002162 taxid:9606 ; # in taxon
          faldo:location ?ensg_location ;
          rdfs:label ?ensg_label ;
          dc:identifier ?ensg_id .
    BIND (strbefore(strafter(str(?ensg_location), "GRCh38/"), ":") AS ?chr)
    FILTER (?chr IN ("1", "2", "3", "4", "5", "6", "7", "8", "9", "10",
                     "11", "12", "13", "14", "15", "16", "17", "18", "19", "20",
                     "21", "22", "X", "Y", "MT" ))
  }
}
```

## `return`
```javascript
//({main, noTf}) => {
async ({tf, geneLabels})=>{
  let tfArray = tf.results.bindings.map(d => d.parent.value.replace("http://identifiers.org/ensembl/", ""));
  async function getTfTargets(tfId) {
    let url = "backend_gene_transcription_factors_chip_atlas"; // parent SPARQLet relative path
    let options = {
      method: 'POST',
      body: 'tfId=' + tfId,
      headers: {
        'Accept': 'application/json',
        'Content-Type': 'application/x-www-form-urlencoded'
      }
    };
    return await fetch(url, options).then(res=>res.json());
  };  

  //let promises = tfArray.map((d) => ({
  //  children: getTfTargets(d),
  //  parent: d
  //}));
  let promises = tfArray.map((d) => {
    return getTfTargets(d);
  });
  let ret = await Promise.all(promises);
  //return ret.reduce(
  //  (objs, current) =>
  //  current.children.reduce(
  //    (x, child) =>
  //    x.concat({parent: current.parent, child: child}),
  //    objs),
  //  []);
  return ret;
}
```
