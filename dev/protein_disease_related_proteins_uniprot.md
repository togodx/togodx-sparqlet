# UniProt keywords 'Disease' classification（守屋）

- Required SPARQLet: backend_protein_uniprot_keywords_uniprot

## Description

- Data sources
    - [UniProt](https://www.uniprot.org/)
    - For details about UniProt Keywords, see [UniProt documentation](https://www.uniprot.org/help/keywords).

- Query
    - Input
        - UniProt ID
    - Output
        - UniProt Keyword in the Disease category
  
## `httpreq`

```javascript
async ({})=>{
  let url = "backend_protein_uniprot_keywords_uniprot"; // parent SPARQLet relative path
  let options = {
    method: 'POST',
    body: 'root=9995',
    headers: {
      'Accept': 'application/json',
      'Content-Type': 'application/x-www-form-urlencoded'
    }
  }
  return await fetch(url, options).then(res=>res.json());
}
```