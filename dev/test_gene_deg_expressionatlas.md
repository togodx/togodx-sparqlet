# DEG in Expression Atlas （池田）

## Description

- Data sources
    - Expression Atlas RDF

## Endpoint

https://integbio.jp/rdf/ebi/sparql

## `main`

```sparql
# Endpoint: https://integbio.jp/rdf/ebi/sparql
PREFIX dc: <http://purl.org/dc/elements/1.1/>
PREFIX obo: <http://purl.obolibrary.org/obo/>
PREFIX taxon: <http://identifiers.org/taxonomy/>
PREFIX ido: <http://identifiers.org/>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX ensembl: <http://rdf.ebi.ac.uk/resource/ensembl/>
PREFIX gxaterms: <http://rdf.ebi.ac.uk/terms/expressionatlas/>
PREFIX gxa:  <http://rdf.ebi.ac.uk/resource/expressionatlas/>
PREFIX dcterms: <http://purl.org/dc/terms/>

SELECT DISTINCT ?dataset ?desc ?analysis ?analysis_type ?analysis_label ?factor
FROM <http://rdf.ebi.ac.uk/dataset/expressionatlas>
WHERE {
  ?dataset gxaterms:hasPart ?analysis ;
           dcterms:description ?desc ;
           obo:RO_0002162 obo:NCBITaxon_9606 .
  VALUES ?analysis_type {gxaterms:MicroarrayDifferentialExpressionAnalysis gxaterms:RNASeqDifferentialAnalysis}
  ?analysis a ?analysis_type ;
            rdfs:label ?analysis_label ;
            gxaterms:hasFactorValue [
              gxaterms:propertyType ?factor_type ;
              a ?factor
            ] .
  VALUES ?factor_type { "DISEASE" }
}
ORDER BY ?factor
#LIMIT 5
```

## `return`
```javascript
async ({main}) => {
  let tree = [{id: "root", label: "root node", root: true},
              {id: "Microarray", label: "Microarray", parent: "root"},
              {id: "RNASeq", label: "RNA-seq", parent: "root"}];
  let datasetSet = new Set();
  let analysisIds = new Array();
  main.results.bindings.forEach((d) => {
    let type = d.analysis_type.value.replace("http://rdf.ebi.ac.uk/terms/expressionatlas/", "").replace(/Differential(Expression)?Analysis/g, "");
    let datasetId = d.dataset.value.replace("http://rdf.ebi.ac.uk/resource/expressionatlas/", "");
    let analysisId = d.analysis.value.replace("http://rdf.ebi.ac.uk/resource/expressionatlas/", "");
    if (!datasetSet.has(datasetId)) {
      tree.push({parent: type, id: datasetId, label: d.desc.value});
      datasetSet.add(datasetId);
    }
    tree.push({parent: datasetId, id: analysisId, label: d.analysis_label.value});
    analysisIds.push(analysisId);
  });
  for (let i = 0; i <= analysisIds.length; i++) {
    let analysisId = analysisIds[i];
    const degs = await fetch('test_backend_gene_deg_expressionatlas', {
      method: 'POST',
      body: `analysisId=${analysisId}`,
      headers: {
        'Accept': 'application/json',
        'Content-Type': 'application/x-www-form-urlencoded'
      }
    }).then((res) => {
      if (res.ok) {
        return res.json();
      } else {
        errors.push(analysisId);
      }
    });
    degs.forEach((deg) => {
      tree.push({parent: analysisId, id: deg.id, label: deg.label, leaf: true});
    });
  }
  return tree;
};
```