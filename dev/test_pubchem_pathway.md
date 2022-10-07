# classification of PubChem_pathway（信定）
## Description
## Endpoint
https://integbio.jp/rdf/pubchem/sparql

## `classification`
```sparql
PREFIX dct:	<http://purl.org/dc/terms/>
PREFIX biopax:	<http://www.biopax.org/release/biopax-level3.owl#>
PREFIX foaf: <http://xmlns.com/foaf/0.1/>
SELECT  distinct ?parent_id ?parent_path_name ?child_id ?child_path_name ?homepage
WHERE {
  values ?homepage {<https://www.pharmgkb.org/> <https://pathbank.org/> <https://reactome.org/>}
  ?parent_path a biopax:Pathway ;
            dct:title  ?parent_path_name ;
            biopax:pathwayComponent ?child_path .
  ?child_path a biopax:Pathway ;
     dct:title  ?child_path_name ;
     dct:source ?source .
  ?source a dct:Dataset ;
         foaf:homepage  ?homepage .
 BIND (strafter(str(?parent_path), "http://rdf.ncbi.nlm.nih.gov/pubchem/pathway/") AS ?parent_id)
 BIND (strafter(str(?child_path), "http://rdf.ncbi.nlm.nih.gov/pubchem/pathway/") AS ?child_id)
}
limit 100
```