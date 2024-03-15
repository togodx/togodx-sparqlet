# PDB # of polypeptide distribution (井手) (10/27-pdb更新版)

## Description
 
- Data sources
    - Number of peptides contained in one PDB entry
    - This item based on the data of January 20, 2021 of PDBj. 
        - The latest data can be obtained from the URL below. https://data.pdbj.org/pdbjplus/data/pdb/rdf/
- Query
    - Input
        - PDB ID
    - Output
        - The number of peptides in each PDB entry.

## Endpoint

https://rdfportal.org/pdb/sparl

* {{SPARQLIST_TOGODX_SPARQL}}

## `withAnnotation`

```sparql
PREFIX dc: <http://purl.org/dc/elements/1.1/>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX pdbo: <http://rdf.wwpdb.org/schema/pdbx-v50.owl#>
PREFIX pdbr: <http://rdf.wwpdb.org/pdb/>
PREFIX tax: <http://purl.uniprot.org/taxonomy/>

SELECT (COUNT(?polypeptide) AS ?value) ?leaf ?label
WHERE {
  VALUES ?parm { "polypeptide(L)"  "polypeptide(D)" } #DNAのentryを排除
  ?leaf a pdbo:datablock ;
        pdbo:has_entityCategory / pdbo:has_entity / pdbo:referenced_by_entity_src_gen / pdbo:link_to_taxonomy_source tax:9606 ;
        dc:title ?label ;
        pdbo:has_entity_polyCategory / pdbo:has_entity_poly ?polypeptide .
  ?polypeptide pdbo:entity_poly.type ?parm .
}
```

## `results`

```javascript
({withAnnotation})=>{
  const idPrefix = "http://rdf.wwpdb.org/pdb/";
  
  return withAnnotation.results.bindings.map(d => {
    return {
      id: d.leaf.value.replace(idPrefix, ""),
      label: d.label.value,
      value: Number(d.value.value),
      binId: Number(d.value.value) + 1,
      binLabel: d.value.value
    }
  });
}
```
