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

SELECT DISTINCT ?tf
WHERE {
  VALUES ?tf { ensembl:ENSG00000275700 ensembl:ENSG00000101544 ensembl:ENSG00000048052 } # for test
  GRAPH <http://rdf.integbio.jp/dataset/togosite/chip_atlas> {
    ?tf obo:RO_0002428 [] .
  }
}#LIMIT 5
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
async ({tf, geneLabels}) => {
  let times = [];
  times.push(new Date());
  let tfArray = tf.results.bindings.map(d => d.tf.value.replace("http://identifiers.org/ensembl/", ""));
  let geneLabelMap = geneLabels.results.bindings.reduce((map, x) => map.set(x.ensg_id.value, x.ensg_label.value), new Map());
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
  times.push(new Date());
  let promises = tfArray.map((d) => getTfTargets(d));
  times.push(new Date());
  let targetsArray = await Promise.all(promises); // [[target genes of tfArray[0]], [target genes of tfArray[1]], ...]
  times.push(new Date());
  let woTfGenes = new Set(geneLabelMap.keys());
  let tree = tfArray.reduce(
    (objs, tf, i) =>
    targetsArray[i].reduce(
      (x, target) => {
        woTfGenes.delete(target);
        return x.concat(
          {
            parent: geneLabelMap.get(tf),
            id: target,
            label: geneLabelMap.get(target),
            leaf: true
          }
        )
      },
      objs.concat(
        {
          parent: "root",
          id: geneLabelMap.get(tf), // use TF labels as ids of TFs to sort by label
          label: geneLabelMap.get(tf)
        }
      )
    ),
    [{id: "root", label: "root node", root: true}]);
  times.push(new Date());
  return times;
  //return targetsArray;
}
```
