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
  const idPrefix = "http://rdf.ebi.ac.uk/resource/ensembl/";
  
  let tree = [
    {
      id: "root",
      label: "root node",
      root: true
    }
  ];

  let edge = {};
  // アノテーション関係
  data.results.bindings.map(d => {
    // 分類ノード id を sortable に
    let parent_id = d.parent.value;
    if (parent_id == "X") parent_id = "23";
    else if (parent_id == "Y") parent_id = "24";
    else if (parent_id == "MT") parent_id = "25";
    else {
      parent_id = ('00' + parent_id).slice(-2);
      d.parent.value = "chr" + d.parent.value;
    }
    tree.push({
      id: d.child.value.replace(idPrefix, ""),
      label: d.child_label.value,
      leaf: true,
      parent: parent_id
    })
    // root との親子関係を追加
    if (!edge[d.parent.value]) {
      edge[d.parent.value] = true;
      tree.push({     
        id: parent_id,
        label: d.parent.value,
        leaf: false,
        parent: "root"
      })
    }
  });
  
  return tree;
};
```