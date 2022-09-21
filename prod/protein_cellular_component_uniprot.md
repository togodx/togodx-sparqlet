# Gene Ontology 'Cellular component' distribution（守屋）

- Required SPARQLet: [backend_protein_go_uniprot](https://togodx.integbio.jp/sparqlist/backend_protein_go_uniprot)

## Description

- Data sources
    - [UniProt](https://www.uniprot.org/)
    
- Query
    - Input
        - UniProt ID
    - Output
        - Gene Ontology terms of biological process domain
  
## `httpreq`

```javascript
async ({})=>{
  let url = "backend_protein_go_uniprot"; // parent SPARQLet relative path
  let options = {
    method: 'POST',
    body: 'root=GO_0005575',
    headers: {
      'Accept': 'application/json',
      'Content-Type': 'application/x-www-form-urlencoded'
    }
  }
  return await fetch(url, options).then(res=>res.json());
}
```