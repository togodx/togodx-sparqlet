# PDBエントリを実験手法で分類 (井手)作業中

## Description
 
- Data sources
    - Experimental methods used for 3D structure acquisition in the PDB entry.
    - This item based on the data of January 20, 2021 of PDBj. 
        - The latest data can be obtained from the URL below. https://data.pdbj.org/pdbjplus/data/pdb/rdf/
- Query
    - Input
        - PDB ID
    - Output
        - The experimental method in each entry.

## Endpoint

https://integbio.jp/togosite/sparql

## `methods`
```sparql

PREFIX pdbr: <https://rdf.wwpdb.org/pdb/>
PREFIX pdbo: <https://rdf.wwpdb.org/schema/pdbx-v50.owl#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX dc: <http://purl.org/dc/elements/1.1/>

SELECT DISTINCT ?leaf ?label ?methods_id                       
   WHERE {
          ?leaf       rdf:type	               pdbo:datablock .
          ?leaf       dc:title  	           ?label .
          ?leaf       pdbo:has_exptlCategory   ?exptlCategory .
          ?exptlCategory  rdf:type                 pdbo:exptlCategory .
          ?exptlCategory  pdbo:has_exptl	       ?exptl .
          ?exptl          pdbo:exptl.method	       ?methods .
          BIND(REPLACE(?methods, " ", "_") AS ?methods_id)
         }

```

## `results`

```javascript
({methods})=>{
  const idPrefix = "https://rdf.wwpdb.org/pdb/";
  
  return methods.results.bindings).map(d => {
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