# DEG in Expression Atlas （池田）

Input a GXA analysis ID.   
By default, returns genes with increased expression in test samples.   
If `decreased` is `true`, returns genes with decreased expression in test samples.

## Description

- Data sources
    - Expression Atlas RDF

## Parameters
* `analysisId` (type: GXA analysis ID)
  * example: E-GEOD-11839#analysis-27169FAC62313C2E1ED00EC7D1B12220
* `decreased` (type: boolean)
  * example: true

## Endpoint

https://integbio.jp/rdf/ebi/sparql

## `main`

```sparql
# Endpoint: https://integbio.jp/rdf/ebi/sparql
PREFIX gxaterms: <http://rdf.ebi.ac.uk/terms/expressionatlas/>

SELECT DISTINCT ?gene
FROM <http://rdf.ebi.ac.uk/dataset/expressionatlas>
WHERE {
  VALUES ?analysis { <http://rdf.ebi.ac.uk/resource/expressionatlas/{{analysisId}}> }
  {{#if decreased}}
  VALUES ?diff { gxaterms:DecreasedDifferentialExpressionRatio }
  {{else}}
  VALUES ?diff { gxaterms:IncreasedDifferentialExpressionRatio }
  {{/if}}
  ?analysis gxaterms:hasOutput [
    gxaterms:refersTo ?gene ;
    a ?diff
  ] .
  ?gene a gxaterms:EnsemblDatabaseReference .
}
```

## `return`
```javascript
({main}) => {
  return main.results.bindings.map(d => d.gene.value.replace("http://rdf.ebi.ac.uk/resource/ensembl/", ""));
};
```