# classification of PubChem_pathway（信定）
## Description
## Endpoint
https://integbio.jp/rdf/pubchem/sparql
## `leaf`
```sparql
PREFIX dct:	<http://purl.org/dc/terms/>
PREFIX biopax:	<http://www.biopax.org/release/biopax-level3.owl#>

SELECT  distinct ?parent_id ?parent_path_name ?child_id ?child_path_name 
WHERE {
  ?parent_path a biopax:Pathway ;
               dct:title  ?parent_path_name ;
               biopax:pathwayComponent ?child_path .
  ?child_path a biopax:Pathway ;
     dct:title  ?child_path_name .
 filter not exists {?child_path biopax:pathwayComponent ?path_comp}
 BIND (strafter(str(?parent_path), "http://rdf.ncbi.nlm.nih.gov/pubchem/pathway/") AS ?parent_id)
 BIND (strafter(str(?child_path), "http://rdf.ncbi.nlm.nih.gov/pubchem/pathway/") AS ?child_id)
}
limit 100

```
