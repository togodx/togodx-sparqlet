# Genes expressed in tissues (Bgee) (池田)

## Description

- Data sources
    - Bgee latest version: [https://bgee.org/?page=sparql](https://bgee.org/?page=sparql)

## Endpoint

https://bgee.org/sparql/

## `leaf`

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

## Endpoint

https://integbio.jp/togosite/sparql

## `graph`
```sparql
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX skos: <http://www.w3.org/2004/02/skos/core#>
PREFIX owl: <http://www.w3.org/2002/07/owl#>
SELECT DISTINCT *
FROM <http://rdf.integbio.jp/dataset/togosite/uberon>
WHERE {
  [] a owl:Class ;
       skos:notation ?child ;
       rdfs:label ?child_label ;
       rdfs:subClassOf [
         skos:notation ?parent ;
         rdfs:label ?parent_label 
       ] .
  FILTER (REGEX(?child, "UBERON"))
}
```

## `return`

```javascript
({graph, leaf}) => {

  let tree = [
    {
      id: "root"
    },
    {
      id: "UBERON:0001062",
      label: "anatomical entity",
      parent: "root"
    },
     {
      id: "UBERON:0000104",
       label: "life cycle",
      parent: "root"
    }
  ];	
  graph.results.bindings.map(d => {
    let anat_id = d.anat_entity.value.replace("http://purl.obolibrary.org/obo/", "");
    tree.push({
      id: d.child.value,
      label: d.child_label.value,
      parent: d.parent.value
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