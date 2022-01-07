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

SELECT DISTINCT ?leaf ?parent ?label 
 FROM <http://rdf.integbio.jp/dataset/togosite/uniprot>
 WHERE {
   ?leaf a up:Protein;
   		up:mnemonic ?label;
   		up:annotation ?annotation .
   ?annotation a up:Catalytic_Activity_Annotation;
   			up:catalyticActivity ?activity .
   ?activity up:enzymeClass ?eccode .
   BIND(SUBSTR(STR(?eccode),32,1) AS ?parent ) 
  #BIND(SUBSTR(STR(?eccode),32,100) AS ?value ) 
  ?leaf up:proteome ?proteome.
  FILTER(REGEX(STR(?proteome), "UP000005640"))
}
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