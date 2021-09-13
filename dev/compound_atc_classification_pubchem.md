# PubChem で薬を薬効で分類（FDA Approved Drugs → WHO ATC Code）(Server対応中未完)（建石, 守屋, 山本）


## Description

- Data sources
	- PubChem-RDF: ftp://ftp.ncbi.nlm.nih.gov/pubchem/RDF/ 
        - Data for nodes linked to ChEMBL or ChEBI retrieved from https://integbio.jp/rdf/dataset/pubchem

- Query
	- Input
  		- PubChem Compound ID 
	- Output
    	- WHO ATC code (https://www.whocc.no/atc_ddd_index/)

## Endpoint

https://integbio.jp/togosite/sparql

## Parameters

## `data`

```sparql
PREFIX sio: <http://semanticscience.org/resource/>
PREFIX obo: <http://purl.obolibrary.org/obo/>
PREFIX compound: <http://rdf.ncbi.nlm.nih.gov/pubchem/compound/CID>
PREFIX concept: <http://rdf.ncbi.nlm.nih.gov/pubchem/concept/ATC_>
PREFIX pubchemv: <http://rdf.ncbi.nlm.nih.gov/pubchem/vocabulary#>
PREFIX dcterms: <http://purl.org/dc/terms/>
PREFIX skos: <http://www.w3.org/2004/02/skos/core#>
SELECT DISTINCT ?cid ?atc ?inn ?parent ?parent_label
    WHERE {
      VALUES ?cid {  compound:3561  compound:6957673  compound:3226  compound:452548  compound:19861  
                     compound:41781  compound:4909  compound:15814656  compound:13342  compound:11597698  
                  }                  
                   
 
      ?attr a sio:CHEMINF_000562 ;
            sio:is-attribute-of ?cid ; 
            sio:has-value  ?inn ;
            dcterms:subject ?atc .
      ?atc skos:broader ?parent ;
  			   a skos:concept .
      ?parent skos:prefLabel  ?parent_label .
} ORDER BY ?parent
```

## `return`
- 整形
```javascript
({data})=>{
  const idVarName = "cid";
  const idPrfix = "http://rdf.ncbi.nlm.nih.gov/pubchem/compound/CID";
  const categoryPrefix = "http://rdf.ncbi.nlm.nih.gov/pubchem/concept/ATC_";
  
  return data.results.bindings.map(d=>{
      var categorycode = d.atc.value.replace(categoryPrefix, "")
      return {
        id: d[idVarName].value.replace(idPrfix, ""), 
        attribute: {
          categoryId: d.atc.value.replace(categoryPrefix, ""), 
          uri: d.atc.value,
          label : capitalize(d.inn.value)
        }
      }
    });
  
  function capitalize(str) {
    return str.charAt(0).toUpperCase() + str.substring(1);
  }
}
```


