# PDB resolution (井手）

## Description

- Data sources
    - The resolution of the PDB entry.
    - This item based on the data of January 20, 2021 of PDBj. 
        - The latest data can be obtained from the URL below. https://data.pdbj.org/pdbjplus/data/pdb/rdf/
- Query
    - Input
        - PDB ID
    - Output
        - The resolution value contained in each entry.

## Endpoint

https://integbio.jp/togosite/sparql

## `withAnnotation`

```sparql
PREFIX pdbo: <http://rdf.wwpdb.org/schema/pdbx-v50.owl#>
PREFIX pdbr: <http://rdf.wwpdb.org/pdb/>
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX dc: <http://purl.org/dc/elements/1.1/>
PREFIX xsd: <http://www.w3.org/2001/XMLSchema#> 

SELECT DISTINCT ?leaf ?value ?label #COUNT(?PDBentry) AS ?count ?Rfactor_index # ?Rfactor ?Rfactor_free ?resolution
  WHERE {
          ?leaf          rdf:type	                pdbo:datablock .
          ?leaf          dc:title  	                ?label .
          ?leaf          pdbo:has_refineCategory	?refineCategory .
          ?refineCategory    pdbo:has_refine	        ?refine .
          # optional{?refine  pdbo:refine.ls_R_factor_obs ?Rfactor .}
           ?refine           pdbo:refine.ls_d_res_high  ?resolution .
           #optional{?refine  pdbo:refine.ls_R_factor_R_free ?Rfactor_free .}
           #BIND(IF(?Rfactor_free = "", "100", ?Rfactor_free) AS ?Rfactor_temp)
           BIND((Round(100*(xsd:decimal(?resolution)))/100) AS ?value)
         }
ORDER BY ?value
```

## `binIDgen`

```sparql
PREFIX pdbo: <http://rdf.wwpdb.org/schema/pdbx-v50.owl#>
PREFIX pdbr: <http://rdf.wwpdb.org/pdb/>
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX dc: <http://purl.org/dc/elements/1.1/>
PREFIX xsd: <http://www.w3.org/2001/XMLSchema#> 

SELECT DISTINCT ?labelseq #COUNT(?PDBentry) AS ?count ?Rfactor_index # ?Rfactor ?Rfactor_free ?resolution
  WHERE {
          ?leaf          rdf:type	                pdbo:datablock .
          ?leaf          dc:title  	                ?label .
          ?leaf          pdbo:has_refineCategory	?refineCategory .
          ?refineCategory   pdbo:has_refine	        ?refine .
          ?refine           pdbo:refine.ls_d_res_high  ?resolution .
           #optional{?refine  pdbo:refine.ls_R_factor_R_free ?Rfactor_free .}
           #BIND(IF(?Rfactor_free = "", "100", ?Rfactor_free) AS ?Rfactor_temp)
           BIND((Round(100*(xsd:decimal(?resolution)))/100) AS ?labelseq)
         }
ORDER BY ?labelseq
```

## `results`

```javascript
({withAnnotation, binIDgen })=>{
  const idPrefix = "http://rdf.wwpdb.org/pdb/";
  let valarray=[];
  let valrank = [];
  let length = Object.keys(binIDgen.results.bindings).length;    //解像度の数値を持つ配列の長さをlengthに代入
  //console.log(length);
  //console.log(binIDgen.results.bindings);
  //console.log(Object.keys(binIDgen.results.bindings).length);
  let i =1;
  binIDgen.results.bindings.map(b => {							//binId,解像度の数値を持つ２次元配列を作成
    valrank=[ i, Number(b.labelseq.value)];
    valarray.push(valrank);
    i++;
  });
  //console.log(valarray);     
  
  return withAnnotation.results.bindings.map(d => {
    return {
      id: d.leaf.value.replace(idPrefix, ""),
      label: d.label.value,
      value: Number(d.value.value),
      binId: binidgen(Number(d.value.value)),
      binLabel: d.value.value
    }
  });
    function binidgen(s) {                                   //解像度からbinIdを導き出す関数を作成する関数
    let target = valarray.filter( e => e[1] === s );
    return String(target[0][0]);
    }
}
```
