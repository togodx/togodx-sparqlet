# ChEMBLをsubstancetypeで分類する（信定・鈴木・八塚・池田）

## Description

- Data sources
    - ChEMBL-RDF: http://ftp.ebi.ac.uk/pub/databases/chembl/ChEMBL-RDF/
- Output
    - Substance type

## Endpoint

https://integbio.jp/rdf/ebi/sparql

## `main`

```sparql
PREFIX chembl: <http://rdf.ebi.ac.uk/terms/chembl#>

SELECT DISTINCT ?type
FROM <http://rdf.ebi.ac.uk/dataset/chembl>
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
      id: d.type.value.replace(" ", ""),
      label: d.type.value,
      parent: 'root'
    });
  });

  let errors = [];
  //for (let i = 0; i <= 9; i++) {
  for (let i = 11111; i <= 11112; i++) {
    const results = await fetch('backend_compound_substance_type_chembl', {
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
        errors.push(`${i}`);
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
