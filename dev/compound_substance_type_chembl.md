# ChEMBLをsubstancetypeで分類する（信定・鈴木・八塚） 作業中（多数問題）
現在sparqlの結果数を制限中

## Description

- Data sources
    - (More data sources description goes here..)
    - ChEMBL-RDF 28.0: http://ftp.ebi.ac.uk/pub/databases/chembl/ChEMBL-RDF/
- Query
    - (More query details go here..)
    -  Input
        - ChEMBL ID
    - Output
        - Substance type
## Parameters
* `off_num` (type: offset number)
  * example: 0, 5, 10
  
## `numArray`
- ユーザが指定したoff_numリストを配列に分割

```javascript
({off_num}) => {
  off_num = off_num.replace(/,/g," ")
   if (off_num.match(/[^\s]/))  return off_num.split(/\s+/);
  return false;
}
```
  
## Endpoint

https://integbio.jp/togosite/sparql

## `data`

```sparql
PREFIX cco: <http://rdf.ebi.ac.uk/terms/chembl#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
SELECT DISTINCT?parent ?child ?child_label
FROM <http://rdf.integbio.jp/dataset/togosite/chembl>
WHERE 
{
  ?substance cco:substanceType ?parent ;
                       cco:chemblId  ?child ;
             rdfs:label ?child_label .
             }
{{#each numArray}}  OFFSET {{ off_num}} {{/each}}
LIMIT 20
```
