# Gene Ontology 'Cellular component' classification（池田）

- Required SPARQLet: backend_gene_go_ncbigene

## Description

- Data sources
    - NCBI Gene
    
- Query
    - Input
        - NCBI Gene ID
    - Output
        - Gene Ontology terms of cellular component domain
  
## `httpreq`

```javascript
async ({})=>{
  let url = "backend_gene_go_ncbigene"; // parent SPARQLet relative path
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