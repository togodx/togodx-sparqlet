# ChEMBLを薬の開発フェーズで分類する（信定） 

## Endpoint

http://sparql-proxy-togodx-1:3000/sparql

## Parameters
* `i` (The first digit of the CHEMBL ID: 1..9)
  * default: 1

## `main`
```sparql
PREFIX cco: <http://rdf.ebi.ac.uk/terms/chembl#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
SELECT DISTINCT?chembl ?parent ?child ?child_label  
FROM <http://rdf.integbio.jp/dataset/togosite/chembl>
WHERE 
{
 ?chembl cco:chemblId ?child ;
            rdfs:label ?child_label ;
            cco:highestDevelopmentPhase ?parent .
  filter not exists { ?chembl a cco:DrugIndication }
  FILTER (regex(str(?chembl), 'CHEMBL{{i}}'))
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