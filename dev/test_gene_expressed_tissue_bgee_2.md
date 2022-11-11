# Genes expressed in tissues (Bgee) (池田, 守屋)

## Parameters

* `obo`
  * default: UBERON_0001296


## `return`
```javascript
async ({obo})=>{
    const options = {
      method: 'POST',
      bodt: 'obo=' + obo,
      headers: {
        'Accept': 'application/json',
        'Content-Type': 'application/x-www-form-urlencoded'
      }
    }
    return await fetch("test_gene_expressed_tissue_bgee_backend", options).then(res=>res.json());
};
```
