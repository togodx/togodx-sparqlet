# ChEMBLを薬の開発フェーズで分類する（信定、池田）

## Description

- Data sources
    - ChEMBL-RDF: http://ftp.ebi.ac.uk/pub/databases/chembl/ChEMBL-RDF/
- Output
    - Highest development phase

## `return`

```javascript
async ({}) => {
  let tree = [
    {
      id: "root",
      label: "root node",
      root: true
    }
  ];

  for (let i = 0; i <= 4; i++) {
  // development_phase にラベルをつける
    let label = "";
    if (i == 0) label = "0: No description";
    else if (i == 1) label = "1: PK tolerability";
    else if (i == 2) label = "2: Efficacy";
    else if (i == 3) label = "3: Safety & Efficacy";
    else if (i == 4) label = "4: Indication Discovery & expansion";

    tree.push({
      id: i,
      label: label,
      parent: 'root'
    });
  };

  return tree;
}
```
