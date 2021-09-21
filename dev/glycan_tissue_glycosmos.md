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

SELECT DISTINCT ?parent_label ?parent ?child ?child_label ?sbsmpt
WHERE {
  [] glycan:has_glycan ?child ;
     a <http://purl.jp/bio/4/id/200906013374193296> ;
     glycan:has_taxon <http://rdf.glycoinfo.org/source/9606> ;
     glycan:has_tissue ?parent .
  ?parent rdfs:label ?parent_label .
  OPTIONAL {
    ?child glycan:has_glycosequence / glycan:has_sequence / ^rdfs:label / a ?sbsmpt .
  }
  OPTIONAL {
    ?child skos:altLabel ?child_label .
  }
}
```

## `all`
差分を取る SPARQL だとなぜか数が合わない。全部取ってから js 内で差分を取る
```sparql
PREFIX xsd: <http://www.w3.org/2001/XMLSchema#>
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX glycan: <http://purl.jp/bio/12/glyco/glycan#>
PREFIX sbsmpt: <http://www.glycoinfo.org/glyco/owl/relation#>
PREFIX dcterms: <http://purl.org/dc/terms/>
PREFIX skos: <http://www.w3.org/2004/02/skos/core#>

SELECT DISTINCT ?gtc ?label ?sbsmpt
FROM <http://rdf.glytoucan.org/partner/glycome-db>
FROM <http://rdf.glytoucan.org/partner/bcsdb>
FROM <http://rdf.glytoucan.org/partner/glycoepitope>
FROM <http://rdf.glycosmos.org/glycans/subsumption>
FROM <http://rdf.glycosmos.org/glycans/seq>
WHERE {
  ?wurcs a ?sbsmpt ;
         rdfs:label / ^glycan:has_sequence / ^glycan:has_glycosequence / skos:altLabel ?label;
         rdfs:seeAlso ?gtc ;
         sbsmpt:subsumes* / dcterms:source / glycan:is_from_source / rdfs:seeAlso <http://identifiers.org/taxonomy/9606> .
  VALUES ?sbsmpt { sbsmpt:Glycosidic_topology sbsmpt:Linkage_defined_saccharide }
}
```

## `return`

```javascript
({data, all}) => {
  const parentIdPrefix = "http://purl.obolibrary.org/obo/";
  const childIdPrefix = "http://rdf.glycoinfo.org/glycan/";

  let tree = [{
    id: "root",
    label: "root node",
    root: true
  }];
  let chk = {};
  const cap = 40;
  function makeLabel(l, sbsmpt, cap) {
    let label = "";
    if (l) {
      label = l;
      if (label.length > cap) {
        label = label.substr(0, cap) + "...";
      }
    } else if (sbsmpt) {
      label = "(" + sbsmpt + ")";
    }
    return label;
  };
  // typed literal とそうでないのとで label が二重についているが、 SPARQL では除けないためここで uniq する
  let uniq = Array.from(
    data.results.bindings.reduce(
      (map, current) =>
      map.set(current.parent_label.value + "-" + current.child.value, current), new Map()).values());
  let unclassified = {};
  all.results.bindings.forEach(d => {
    unclassified[d.gtc.value.replace("https://glytoucan.org/Structures/Glycans/", "")] = {
      label: d.label.value,
      sbsmpt: d.sbsmpt.value
    };
  });
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

    tree.push({
      id: d.child.value.replace(childIdPrefix, ""),
      label: makeLabel(d.child_label?.value, d.sbsmpt?.value, cap),
      leaf: true,
      parent: d.parent.value.replace(parentIdPrefix, "")
    });
    delete unclassified[d.child.value.replace(childIdPrefix, "")];
  });

  tree.push({
    id: "unclassified",
    label: "N/A",
    leaf: false,
    parent: "root"
  });

  Object.keys(unclassified).forEach(d => {
    tree.push({
      id: d.replace("https://glytoucan.org/Structures/Glycans/", ""),
      label: makeLabel(unclassified[d].label, unclassified[d].sbsmpt, cap),
      leaf: true,
      parent: "unclassified"
    });
  });

  return tree;
};
```