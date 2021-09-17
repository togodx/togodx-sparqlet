# ChEMBLをsubstancetypeで分類する（信定・鈴木・八塚）

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

## `main`

```sparql
PREFIX chembl: <http://rdf.ebi.ac.uk/terms/chembl#>

SELECT DISTINCT ?type
FROM <http://rdf.integbio.jp/dataset/togosite/chembl>
WHERE 
{
  ?chembl chembl:substanceType ?type .
}
```

## `return`
```javascript
async ({ main }) => {  
  let tree = [
    {
      id: "root",
      label: "root node",
      root: true
    }
  ];

  main.results.bindings.map(d => {
    tree.push({
      id: d.type.value,
      label: d.type.value,
      parent: 'root'
    });
  });

  let errors = [];
  for (let i = 1; i <= 9; i++) {
    const results = await fetch('backend_compound_substance_type_chembl',　{
      method: 'POST',
      body: `i=${i}`,
      headers: {
        'Accept': 'application/json',
        'Content-Type': 'application/x-www-form-urlencoded'
      }
    }).then((res) => {
      if (res.ok) {
        return res.json();
      } else {
        errors.push(`CHEMBL${i}`);
      }
    });
    
    if (results) {
      results.forEach((elem) => {
        tree.push(elem);
      });
    }
  }

  if (errors.length) {
    tree.push({ errors: errors });
  }
  return tree;
};
```
