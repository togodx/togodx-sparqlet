# ChEMBLをsubstancetypeで分類する

## Endpoint

https://integbio.jp/togosite/sparql

## Parameters
* `i` (The first digit of the CHEMBL ID: 1..9)
  * default: 1

## `main`
```sparql
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX chembl: <http://rdf.ebi.ac.uk/terms/chembl#>

SELECT DISTINCT ?chembl ?type ?id ?label
FROM <http://rdf.integbio.jp/dataset/togosite/chembl>
WHERE {
  ?chembl chembl:substanceType ?type ;
      chembl:chemblId ?id ;
      rdfs:label ?label .
  FILTER (regex(str(?chembl), 'CHEMBL{{i}}'))
}
```

## `return`

```javascript
({ main }) => {
  let tree = [];

  main.results.bindings.forEach((elem) => {
    tree.push({
      id: elem.id.value,
      label: elem.label.value,
      leaf: true,
      parent: elem.type.value
    })
  });

  return tree;
};
```
