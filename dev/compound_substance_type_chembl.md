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
        
## `return`
```javascript
async () => {
  async function getSubstance(off_num) {
    let url = "backend_compound_substance_type_chembl"; // parent SPARQLet relative path
    let options = {
      method: 'POST',
      body: 'off_num=' + off_num,
      headers: {
        'Accept': 'application/json',
        'Content-Type': 'application/x-www-form-urlencoded'
      }
    };
    return await fetch(url, options);
  };  


```