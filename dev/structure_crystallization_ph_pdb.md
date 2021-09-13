# PDBエントリを結晶化時のpHで分類（井手）作業中

## Description
 
- Data sources
    - The pH of crystal formation in the PDB entry
    - This item based on the data of January 20, 2021 of PDBj. 
        - The latest data can be obtained from the URL below. https://data.pdbj.org/pdbjplus/data/pdb/rdf/
- Query
    - Input
        - PDB ID
    - Output
        - The pH of crystal formation in each PDB entry.

## Endpoint

https://integbio.jp/togosite/sparql

## `pH`

```sparql
PREFIX pdbo: <https://rdf.wwpdb.org/schema/pdbx-v50.owl#>
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX pdbr: <https://rdf.wwpdb.org/pdb/>
PREFIX dc: <http://purl.org/dc/elements/1.1/>
PREFIX xsd: <http://www.w3.org/2001/XMLSchema#> 

{{#if mode}}
  SELECT ?pH ?pH_str ?PDBentry ?title
{{else}}
  SELECT ?pH COUNT(?pH) AS ?count 
{{/if}}  
    WHERE {
      {{#if filter_list}} VALUES ?PDBentry { {{#each filter_list}} pdbr:{{this}} {{/each}} } {{/if}}
      {{#if pH_range}}    VALUES (?pH_MIN ?pH_MAX) {( {{#each pH_range}}  {{this}}  {{/each}} )} {{/if}}  
     ?PDBentry             rdf:type	                            pdbo:datablock .
     ?PDBentry             dc:title  	                        ?title .
     ?PDBentry             pdbo:has_exptl_crystal_growCategory	?crystal_growCategory .
     ?crystal_growCategory pdbo:has_exptl_crystal_grow	        ?crystal_grow .
     ?crystal_grow         pdbo:exptl_crystal_grow.pH	        ?pH_str .
     {{#if pH_range}} FILTER(xsd:float(?pH_str) <= xsd:float(?pH_MAX))    
                      FILTER(xsd:float(?pH_str) >= xsd:float(?pH_MIN)) {{/if}}          
     BIND(Round(10*(xsd:decimal(?pH_str))/10) AS ?pH)         
     }
order by ?pH  
```

## `results`

```javascript
({mode, pH})=>{
   if (mode == "objectList") return pH.results.bindings.map(d=>{ 
     return {
       id: d.PDBentry.value.replace("https://rdf.wwpdb.org/pdb/", ""), 
         attribute: {
         categoryId: d.pH_str.value, 
         label: "pH " + d.pH_str.value
         }
       };
    });
   if (mode == "idList") return Array.from(new Set(pH.results.bindings.map(d=>d.PDBentry.value.replace("https://rdf.wwpdb.org/pdb/", "")))); // unique
    return pH.results.bindings.map(d=>{
     return {
       categoryId: d.pH.value, 
       label: "pH " + d.pH.value, 
       count: Number(d.count.value)
       };
   });	
}
```
