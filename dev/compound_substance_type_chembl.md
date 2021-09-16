# ChEMBLをsubstancetypeで分類する（信定・鈴木・八塚） 作業中（多数問題）
現在sparqlの結果数を制限中

## Description

- Data sources
    - (More data sources description goes here..)
    - ChEMBL-RDF 28.0: http://ftp.ebi.ac.uk/pub/databases/chembl/ChEMBL-RDF/
- Query
    - (More query details go here..)
    -  Input
        - ChEMBL ID
    - Output
        - Substance type
        
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


```