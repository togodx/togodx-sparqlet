# UniProt disulfide bond count distribution（守屋）

- require : [backend_protein_number_of_uniprot_annotation_uniprot](./backend_protein_number_of_uniprot_annotation_uniprot)
## Description

- Data sources
    - [UniProt](https://www.uniprot.org/)

- Query
    - Input
        - UniProt ID
    - Output
        - The number of disulfide bond

## `httpreq`

```javascript
async ({})=>{
  let url = "backend_protein_number_of_uniprot_annotation_uniprot"; // parent SPARQLet relative path
  let options = {
    method: 'POST',
    body: 'type=Disulfide_Bond_Annotation',
    headers: {
      'Accept': 'application/json',
      'Content-Type': 'application/x-www-form-urlencoded'
    }
  }
  return await fetch(url, options).then(res=>res.json());
}
```