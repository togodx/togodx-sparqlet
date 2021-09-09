# WIP: ChEBI Application での分類 （川島、建石）→ Server対応中、未完
- 分類系
- Root は Application（CHEBI:33232）
## Description

- Data sources
    -  [Chemical Entities of Biological Interest (ChEBI) ](https://www.ebi.ac.uk/chebi/) 
- Query
    - Input
        - ChEBI id (number) for chemical compound(s)
    - Output
        -  ChEBI id (number) for application type(s) (subcategories of [Application (CHEBI:33232)](https://www.ebi.ac.uk/chebi/searchId.do?chebiId=CHEBI:33232) ) corresponding to the compound(s)
- Supplementary Information
	-  The classification of compounds according to their application, defined in ChEBI ontology.
	- ChEBI Ontologyに定義された用途による、	化合物の分類です。
    
## Parameters 

* `queryIds` 化合物のID（数字のみ）またはIDのリスト
  * default: 
  * example: 18012, 27732, 17594, 16866, 46195, 62867   


## `queryArray` 
- いらないが、テスト用においとく(後で消す）
- ユーザが指定した ID リストを配列に分割

```javascript
({queryIds}) => {
  queryIds = queryIds.replace(/,/g," ")
   if (queryIds.match(/[^\s]/))  return queryIds.split(/\s+/);
  return false;
}
```

## Endpoint

https://integbio.jp/togosite/sparql

## `leaf`
 - compoundに Applicationをくっつける
```sparql
PREFIX obo: <http://purl.obolibrary.org/obo/>
PREFIX owl: <http://www.w3.org/2002/07/owl#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX CHEBI: <http://purl.obolibrary.org/obo/CHEBI_>


SELECT distinct ?compound GROUP_CONCAT(DISTINCT ?label; SEPARATOR = ", ") as ?label 
                          ?application                                      
                          GROUP_CONCAT(DISTINCT ?application_label; SEPARATOR = ", ") AS ?application_label
FROM <http://rdf.integbio.jp/dataset/togosite/chebi>
WHERE 
{
  {{#if queryArray}}
    VALUES ?compound { {{#each queryArray}} CHEBI:{{this}} {{/each}} }
  {{/if}}
  
  VALUES ?application {obo:CHEBI_33232}
      
  ?compound a owl:Class ;
    rdfs:label ?label ;
    rdfs:subClassOf ?r .
  ?r a owl:Restriction ;
    owl:onProperty obo:RO_0000087 ;
    owl:someValuesFrom ?role .
  ?role rdfs:subClassOf* ?application .
  ?application rdfs:label ?application_label .
  optional{
  	?x rdfs:subClassOf ?application.
  }
}


```



## `return`
- 整形
```javascript
({ leaf})=>{
  const idVarName = "compound";
  const idLabelVarName = "label";
  const categoryVarName = "application";
  const categoryLabelVarName = "application_label";
  
  const idPrefix = "http://purl.obolibrary.org/obo/CHEBI_";
  const categoryPrefix = "http://purl.obolibrary.org/obo/CHEBI_";
  
  return data.results.bindings.map(d=>{
    return {
      id: d[idVarName].value.replace(idPrefix, ""), 
      attribute: {
        categoryId: d[categoryVarName].value.replace(categoryPrefix, ""), 
        uri: d[categoryVarName].value,
        label : d[categoryLabelVarName].value
      }
    }
  });
  	
}
```