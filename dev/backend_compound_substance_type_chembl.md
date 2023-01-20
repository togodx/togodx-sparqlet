# ChEMBLをsubstancetypeで分類する

## Endpoint

https://integbio.jp/rdf/ebi/sparql

## Parameters
* `i` (The last digit of the CHEMBL ID: 0..9)

## `main`
```sparql
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX chembl: <http://rdf.ebi.ac.uk/terms/chembl#>

SELECT DISTINCT ?chembl ?parent ?child ?child_label
FROM <http://rdf.ebi.ac.uk/dataset/chembl>
WHERE {
  ?chembl chembl:substanceType ?parent ;
          chembl:chemblId ?child ;
          rdfs:label ?child_label .
  FILTER(STRENDS(?child, '{{i}}'))
}
```

## `return`

```javascript
({ main }) => {
  let tree = [];

  main.results.bindings.forEach((elem) => {
    tree.push({
      id: elem.child.value,
      label: elem.child_label.value,
      leaf: true,
      parent: elem.parent.value.replace(" ", "")
    })
  });

  return tree;
};
```
