# Genes expressed in tissues (Bgee) (池田)

## Description

- Data sources
    - Bgee latest version: [https://bgee.org/?page=sparql](https://bgee.org/?page=sparql)

## Endpoint

https://bgee.org/sparql/

## `data`

```sparql
PREFIX orth: <http://purl.org/net/orth#>
PREFIX up: <http://purl.uniprot.org/core/>
PREFIX genex: <http://purl.org/genex#>
PREFIX obo: <http://purl.obolibrary.org/obo/>
PREFIX dcterms: <http://purl.org/dc/terms/>

SELECT DISTINCT ?gene_id ?anat_entity {
    ?gene a orth:Gene ;
          orth:organism/obo:RO_0002162 <http://purl.uniprot.org/taxonomy/9606> ;
          dcterms:identifier ?gene_id ;
          genex:isExpressedIn ?anat_entity .
    ?anat_entity a genex:AnatomicalEntity .
    FILTER (?anat_entity != obo:UBERON_0001062 && ?anat_entity != obo:UBERON_0000061 
&& ?anat_entity != obo:UBERON_0000465 && ?anat_entity != obo:UBERON_0000468 && ?anat_entity != obo:UBERON_0000467)
}
```

## `return`

```javascript
({data}) => {  
  let tree = [{id: "root", label: "root node", root: true}];

  let anats = {};
  data.results.bindings.map(d => {
    let anat_id = d.anat_entity.value.replace("http://purl.obolibrary.org/obo/", "");
    tree.push({
      id: d.gene_id.value,
      label: d.gene_name.value,
      leaf: true,
      parent: anat_id
    })
    if (!anats[anat_id]) {
      anats[anat_id] = true;
      tree.push({     
        id: anat_id,
        label: d.anat_name.value,
        parent: "root"
      })
    }
  });
  
  return tree;
};
```