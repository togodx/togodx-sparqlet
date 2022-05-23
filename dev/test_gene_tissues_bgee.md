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

SELECT DISTINCT ?gene_id ?gene_name ?anat_entity ?anat_name {
    ?gene a orth:Gene ;
          rdfs:label ?gene_name ;
          dcterms:identifier ?gene_id ;
          genex:isExpressedIn ?cond .
    ?cond genex:hasAnatomicalEntity ?anat_entity .
    ?anat_entity rdfs:label ?anat_name .
    ?cond obo:RO_0002162 <http://purl.uniprot.org/taxonomy/9606> . 
}
#LIMIT 200
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