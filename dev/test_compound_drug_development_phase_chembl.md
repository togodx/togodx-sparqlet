# test_ChEMBLを薬の開発フェーズで分類する（信定） 

## Description

- Data sources
    - (More data sources description goes here..)
    - ChEMBL-RDF 28.0: http://ftp.ebi.ac.uk/pub/databases/chembl/ChEMBL-RDF/
- Query
    - (More query details go here..)
    -  Input
        - ChEMBL ID
    - Output
        - Highest development phase
        
 ## Endpoint

https://integbio.jp/togosite/sparql

## `main`

```sparql
PREFIX cco: <http://rdf.ebi.ac.uk/terms/chembl#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
SELECT DISTINCT?chembl_id ?chembl_label ?development_phase
FROM <http://rdf.integbio.jp/dataset/togosite/chembl>
WHERE 
{
  ?chembl cco:chemblId ?chembl_id ;
            rdfs:label ?chembl_label ;
            cco:highestDevelopmentPhase ?development_phase .
  filter not exists { ?chembl a cco:DrugIndication }
  filter  (?development_phase !="0" )
}
```

## `allLeaf`
- 全ChEMBL (without annotation 用)
```sparql
PREFIX cco: <http://rdf.ebi.ac.uk/terms/chembl#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
SELECT DISTINCT?chembl_id ?chembl_label
FROM <http://rdf.integbio.jp/dataset/togosite/chembl>
WHERE 
{
  ?chembl cco:chemblId ?chembl_id ;
          rdfs:label ?chembl_label ;
          cco:highestDevelopmentPhase ?development_phase .
  filter not exists { ?chembl a cco:DrugIndication }
}
```

## `return`

```javascript
({main}) => {
  let tree = [
    {
      id: "root",
      label: "root node",
      root: true
    }
  ];

  main.results.bindings.map(d => {
  // development_phase にラベルをつける
    let parent_label = d.development_phase.value;
    if (parent_label  == 0) parent_label = "0: No description";
    else if (parent_label  == 1) parent_label = "1: PK tolerability";
    else if (parent_label  == 2) parent_label = "2: Efficacy";
    else if (parent_label  == 3) parent_label = "3: Safety & Efficacy";
    else if  (parent_label  == 4) parent_label = "4: Indication Discovery & expansion";
  
    let edge = {};
  data.results.bindings.map(d => {
    tree.push({
      id: d.child.value,
      label: d.child_label.value,
      leaf: true,
      parent: d.parent.value
    })
    
    tree.push({
      id: d.development_phase.value,
      label: parent_label,
      parent: 'root'
    });
  });

  let errors = [];
  for (let i = 1; i <= 9; i++) {
    const results = await fetch('backend_compound_drug_development_phase_chembl',　{
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