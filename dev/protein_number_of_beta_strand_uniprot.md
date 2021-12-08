# UniProt beta strand structure count distribution（守屋）

## Description

- Data sources
    - [UniProt](https://www.uniprot.org/)

- Query
    - Input
        - UniProt ID
    - Output
        - The number of beta strand

## `httpreq`

```javascript
async ({})=>{
  let url = "background_protein_number_of_uniprot_annotation_uniprot"; // parent SPARQLet relative path
  let options = {
    method: 'POST',
    body: 'type=Beta_Strand_Annotation',
    headers: {
      'Accept': 'application/json',
      'Content-Type': 'application/x-www-form-urlencoded'
    }
  }
  return await fetch(url, options).then(res=>res.json());
}
```
