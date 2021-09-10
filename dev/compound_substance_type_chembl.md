# ChEMBLをsubstancetypeで分類する（信定・鈴木・八塚） Server対応中未完


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

## Endpoint

https://integbio.jp/togosite/sparql

## `data`
- SPARQL
  - 内訳返す場合とchembl compoundリストを返す場合を handlbars で条件分岐
  - substance type、chembl compoundリストでのフィルタリング

```sparql
prefix cco: <http://rdf.ebi.ac.uk/terms/chembl#>
{{#if mode}}
SELECT DISTINCT ?chembl_id ?substancetype ?type_id
{{else}}
SELECT  ?type_id (COUNT(distinct ?substance) AS ?count)
{{/if}}
WHERE 
{
{{#if query_array}}
  VALUES ?chembl_id { {{#each query_array}} "{{this}}" {{/each}} }
{{/if}}
{{#if category_array}}
  VALUES (?type_id ?substancetype) { {{#each category_array}} ("{{this.cid}}:{{this.type}}" "{{this.type}}") {{/each}} }
{{/if}}
  ?substance cco:chemblId ?chembl_id ;
            cco:substanceType ?substancetype .
}
{{#unless mode}}
group by ?type_id ?count
ORDER BY Desc(?count)
{{/unless}}
```