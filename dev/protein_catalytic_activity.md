# UniProt catalytic_activity（井手）* 220107-作業中

## Description

- Data sources
    - [UniProt](https://www.uniprot.org/)

- Query
    - Input
        - UniProt ID
    - Output
        - The Catalytic activity based on EC number

## Endpoint
https://integbio.jp/togosite/sparql

## `withAnnotation`
```sparql
PREFIX up: <http://purl.uniprot.org/core/>
PREFIX upid: <http://purl.uniprot.org/uniprot/>
PREFIX taxon: <http://purl.uniprot.org/taxonomy/>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX xsd: <http://www.w3.org/2001/XMLSchema#> 

SELECT DISTINCT ?uniprot ?mnemonic ?ecclass ?ecclass1 ?ecclass2 ?ec_sub ?ec_sub_int #?eccode ?annotation
       #SELECT DISTINCT COUNT(?ec_sub) AS ?count ?ec_sub
 FROM <http://rdf.integbio.jp/dataset/togosite/uniprot>
 WHERE {
   ?uniprot a up:Protein ;
            up:mnemonic ?mnemonic;
            up:proteome ?proteome;
            up:annotation ?annotation .
   ?annotation a up:Catalytic_Activity_Annotation;
            up:catalyticActivity/up:enzymeClass ?eccode .
   BIND(SUBSTR(STR(?eccode),32,99) AS ?ecclass) 
   BIND(SUBSTR(?ecclass, 1, STRLEN(?ecclass)-STRLEN(STRAFTER(STRAFTER(?ecclass ,"."),"."))-1) AS ?ec_sub)
   
   BIND(xsd:INTEGER(SUBSTR(?ecclass,1,1))*100 AS ?ecclass1)
   BIND(xsd:INTEGER(STRAFTER(?ec_sub,".")) AS ?ecclass2)
   BIND((?ecclass1+?ecclass2) AS ?ec_sub_int)

   FILTER(REGEX(STR(?proteome), "UP000005640"))
}
#Order by ?ec_sub
limit 10
```

## `results`

```javascript
({withAnnotation})=>{
  const idPrefix = "http://purl.uniprot.org/uniprot/";

  let tree = [
    {
      id: "root",
      label: "root node",
      root: true
    },  
    {
    "id": "01",
    "label": "Oxidoreductases",
    "leaf": false,
    "parent": "root"                    
    },
    {  
    "id": "02",
    "label": "Transferases",
    "leaf": false,
    "parent": "root"                    
    },
    {  
    "id": "03",
    "label": "Hydrolases",
    "leaf": false,
    "parent": "root"                    
    },
    {  
    "id": "04",
    "label": "Lyases",
    "leaf": false,
    "parent": "root"                    
    },
    {  
    "id": "05",
    "label": "Isomerases",
    "leaf": false,
    "parent": "root"                    
    },
    {  
    "id": "06",
    "label": "Ligase",
    "leaf": false,
    "parent": "root"                    
    },
    {  
    "id": "07",
    "label": "Translocase",
    "leaf": false,
    "parent": "root"                    
    }     
    { "id": "101", "label": "Acting on the CH-OH group of donors", "leaf": false, "parent": "01" } 
    { "id": "102", "label": "Acting on the aldehyde or oxo group of donors", "leaf": false, "parent": "01" } 
    { "id": "103", "label": "Acting on the CH-CH group of donors",           "leaf": false, "parent": "01" } 
    { "id": "104", "label": "Acting on the CH-NH(2) group of donors",        "leaf": false, "parent": "01" } 
    { "id": "105", "label": "Acting on the CH-NH group of donors",           "leaf": false, "parent": "01" } 
    { "id": "106", "label": "Acting on NADH or NADPH",                       "leaf": false, "parent": "01" } 
    { "id": "107", "label": "Acting on other nitrogenous compounds as donors", "leaf": false, "parent": "01" } 
    { "id": "108", "label": "Acting on a sulfur group of donors", "leaf": false, "parent": "01" } 
    { "id": "110", "label": "Acting on diphenols and related substances as donors", "leaf": false, "parent": "01" } 
    { "id": "111", "label": "Acting on a peroxide as acceptor", "leaf": false, "parent": "01" } 
    { "id": "113", "label": "Acting on single donors with incorporation of molecular oxygen (oxygenases). The oxygen incorporated need not be derived from O(2)", "leaf": false, "parent": "01" } 
    { "id": "114", "label": "Acting on paired donors, with incorporation or reduction of molecular oxygen. The oxygen incorporated need not be derived from O(2)", "leaf": false, "parent": "01" } 
    { "id": "115", "label": "Acting on superoxide as acceptor", "leaf": false, "parent": "01" } 
    { "id": "116", "label": "Oxidizing metal ions", "leaf": false, "parent": "01" } 
    { "id": "117", "label": "Acting on CH or CH(2) groups", "leaf": false, "parent": "01" } 
    { "id": "118", "label": "Acting on iron-sulfur proteins as donors", "leaf": false, "parent": "01" } 
    { "id": "120", "label": "Acting on phosphorus or arsenic in donors", "leaf": false, "parent": "01" } 
    { "id": "121", "label": "Catalyzing the reaction X-H + Y-H = 'X-Y'", "leaf": false, "parent": "01" } 
  ];
  
  withAnnotation.results.bindings.map(d => {
	let parent_id = d.parent.value;
 	if (parent_id  == "1") {parent_id = "01";}
	else if (parent_id == "2") {parent_id = "02";}
	else if (parent_id == "3") {parent_id = "03";}
	else if (parent_id == "4") {parent_id = "04";}
	else if (parent_id == "5") {parent_id = "05";}
	else if (parent_id == "6") {parent_id = "06";}
	else if (parent_id == "7") {parent_id = "07";}
	else {
       parent_id = "8"; 
    }
    tree.push({
      id: d.leaf.value.replace(idPrefix, ""),
      label: d.label.value,
      leaf: true,
      parent: parent_id
    })
  });
  return tree;
}
```