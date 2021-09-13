# PubChem で薬を薬効で分類（FDA Approved Drugs → WHO ATC Code）(Server対応中未完)（建石, 守屋, 山本）

-  atc_classification_haschild （objectList はOK）を idList対応に修正 (2021/3/24)
-  Filterの効率化 (2021/4/2)
- Parameters:
	- categoryIds:  WHO ATC code (https://www.whocc.no/atc_ddd_index/) or EMPTY
    - queryIds: PubChem Compound ID or EMPTY
    - mode: 'idList' or 'objectList' or EMPTY

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

SELECT DISTINCT ?cid ?category ?label
WHERE {
      # test     
      VALUES ?cid {  compound:3561  compound:6957673  compound:3226  compound:452548  compound:19861  
                     compound:41781  compound:4909  compound:15814656  compound:13342  compound:11597698  
                     compound:3396  compound:60937  compound:86767262  compound:43507  compound:3342  
                     compound:4642  compound:5311497  compound:3356  compound:37464  compound:5353853
                  }

 
      ?attr a sio:CHEMINF_000562 ;
            sio:is-attribute-of ?cid ; 
            sio:has-value  ?WHO_INN ;
            dcterms:subject ?WHO_ATC .
      ?WHO_ATC skos:broader* ?category ;
  			   a skos:concept .
      # filter(strstarts(str(?WHO_ATC), "http://rdf.")) # MeSH IDが入っていることがあるのを捨てる。
                                                      # ATCコード  http://rdf.ncbi.nlm.nih.gov/pubchem/concept/ATC_xxxx
                                                      # MeSH ID    http://id.nlm.nih.gov/mesh/Mxxxxxx
      
      # top。    
      #filter(strlen(str(?category)) = 49)     # regex(str(?category),http://rdf.ncbi.nlm.nih.gov/pubchem/concept/ATC_[A-Z]$)   
  
      ?category skos:prefLabel  ?label .
} ORDER BY ?category

```

## `return`
- 整形
```javascript
({data})=>{
  const idVarName = "cid";
  const idPrfix = "http://rdf.ncbi.nlm.nih.gov/pubchem/compound/CID";
  const categoryPrefix = "http://rdf.ncbi.nlm.nih.gov/pubchem/concept/ATC_";
  
  return data.results.bindings.map(d=>{
      var categorycode = d.category.value.replace(categoryPrefix, "")
      return {
        id: d[idVarName].value.replace(idPrfix, ""), 
        attribute: {
          categoryId: d.category.value.replace(categoryPrefix, ""), 
          uri: d.category.value,
          label : capitalize(d.label.value)+" ("+categorycode+")"
        }
      }
    });
  
  function capitalize(str) {
    return str.charAt(0).toUpperCase() + str.substring(1);
  }
}
```


