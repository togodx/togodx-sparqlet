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

## `tfclassSp`
```sparql
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX tfclass: <http://sybig.de/tfclass#>
PREFIX owl: <http://www.w3.org/2002/07/owl#>

SELECT DISTINCT ?ensg_id ?label ?molsp ?parent
FROM <http://rdf.integbio.jp/dataset/togosite/tfclass>
WHERE { 
  ?s tfclass:xref ?ensg .
  FILTER(STRSTARTS(?ensg, "ENSEMBL_GeneID:"))
  BIND(STRAFTER(?ensg, "ENSEMBL_GeneID:") as ?ensg_id)
  ?s rdfs:subClassOf tfclass:tf_9606 ;
     rdfs:label ?label ;
     rdfs:subClassOf / owl:someValuesFrom ?molsp .
  ?molsp rdfs:label ?molsp_label .
  ?molsp rdfs:subClassOf ?parent .
  FILTER(!isBlank(?parent))
}
```

## `tfclassSp`
```sparql
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX tfclass: <http://sybig.de/tfclass#>
PREFIX owl: <http://www.w3.org/2002/07/owl#>

SELECT DISTINCT ?child ?parent ?child_label ?parent_label
FROM <http://rdf.integbio.jp/dataset/togosite/tfclass>
WHERE { 
  ?s tfclass:xref ?ensg .
  FILTER(STRSTARTS(?ensg, "ENSEMBL_GeneID:"))
  BIND(STRAFTER(?ensg, "ENSEMBL_GeneID:") as ?ensg_id)
  ?s rdfs:subClassOf tfclass:tf_9606 ;
     rdfs:subClassOf / owl:someValuesFrom ?molsp .
  ?molsp rdfs:subClassOf* ?child .
  ?child rdfs:subClassOf ?parent ;
         rdfs:label ?child_label .
  ?parent rdfs:label ?parent_label .
  FILTER(!isBlank(?parent))
}
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
  let tfArray = tf.results.bindings.map(d => d.tf.value.replace("http://identifiers.org/ensembl/", ""));
  let geneLabelMap = new Map();
  geneLabels.results.bindings.forEach((x) => geneLabelMap.set(x.ensg_id.value, x.ensg_label.value));
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

  let tree = [{id: "root", label: "root node", root: true}];
  tfArray.forEach((tf, i) => {
    tree.push(
      {
        parent: "root",
        id: geneLabelMap.get(tf), // use TF labels as ids of TFs to sort by label
        label: geneLabelMap.get(tf)
      });
    targetsArray[i].forEach((target) => {
      woTfGenes.delete(target);
      tree.push(
        {
          parent: geneLabelMap.get(tf),
          id: target,
          label: geneLabelMap.get(target),
          leaf: true
        });
      return;
    });
    return;
  })
  tree.push({parent: "root", id: "unclassified", label: "no known upstream TF"});
  woTfGenes.forEach((gene) => {
    tree.push(
      {
        parent: "unclassified",
        id: gene,
        label: geneLabelMap.get(gene),
        leaf: true
      });
  });
  times.push(new Date());
  //return times;
  return tree;
}
```
