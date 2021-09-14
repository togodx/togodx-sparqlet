# Glycans in tissue （山本・池田）

## Description

- Data sources
    - [GlyCosmos](https://glycosmos.org/data)

- Query
    - Input
        - GlyTouCan ID
    - Output
        - UBERON ID of tissues

## Endpoint

https://ts.glycosmos.org/sparql

## `data`

```sparql
PREFIX xsd: <http://www.w3.org/2001/XMLSchema#>
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX info: <http://rdf.glycoinfo.org/glycan/>
PREFIX glycan: <http://purl.jp/bio/12/glyco/glycan#>
PREFIX obo: <http://purl.obolibrary.org/obo/>
PREFIX skos: <http://www.w3.org/2004/02/skos/core#>

SELECT DISTINCT ?parent_label ?parent ?child ?child_label
WHERE {
  [] glycan:has_glycan ?child ;
     a <http://purl.jp/bio/4/id/200906013374193296> ;
     glycan:has_taxon <http://rdf.glycoinfo.org/source/9606> ;
     glycan:has_tissue ?parent .
  ?parent rdfs:label ?parent_label .
  OPTIONAL {
    ?child skos:altLabel ?child_label .
  }
}
```

## `return`

```javascript
({data}) => {
  const parentIdPrefix = "http://purl.obolibrary.org/obo/";
  const childIdPrefix = "http://rdf.glycoinfo.org/glycan/";

  let tree = [{
    id: "root",
    label: "root node",
    root: true
  }];
  let chk = {};
  let cap = 20;
  // typed literal とそうでないのとで label が二重についているが、 SPARQL では除けないためここで uniq する
  let uniq = Array.from(
    data.results.bindings.reduce(
      (map, current) => 
      map.set(current.parent_label.value + "-" + current.child.value, current), new Map()).values());
  uniq.forEach(d => {
    if (!chk[d.parent.value]) {
      chk[d.parent.value] = true;
      tree.push({     
        id: d.parent.value.replace(parentIdPrefix, ""),
        label: d.parent_label.value,
        leaf: false,
        parent: "root"
      })
    }
    let child_label = "";
    if (d.child_label.value) {
      child_label = d.child_label.value;
      if (child_label.length > cap) {
        child_label = child_label.substr(0, cap) + "...";
      }
    }
    tree.push({
      id: d.child.value.replace(childIdPrefix, ""),
      label: child_label,
      leaf: true,
      parent: d.parent.value.replace(parentIdPrefix, "")
    });
    return;
  });
  
  return tree;
};
```