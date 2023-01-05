# ChEMBLを薬の開発フェーズで分類する（信定、池田）

## Endpoint

https://integbio.jp/rdf/ebi/sparql

## Parameters
* `i` (The last digit of CHEMBL IDs: 0..9)

## `main`
```sparql
PREFIX cco: <http://rdf.ebi.ac.uk/terms/chembl#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>

SELECT DISTINCT ?chembl ?parent ?child ?child_label
FROM <http://rdf.ebi.ac.uk/dataset/chembl>
WHERE
{
  ?chembl cco:chemblId ?child ;
          rdfs:label ?child_label ;
          cco:highestDevelopmentPhase ?parent .
  FILTER NOT EXISTS { ?chembl a cco:DrugIndication }
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
      parent: elem.parent.value,
   })
  });

  return tree;
};
```
