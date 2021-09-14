# Glycan subsumption （山本・池田）

## Description

- Data sources
    - [GlyCosmos](https://glycosmos.org/data)

- Query
    - Input
        - GlyTouCan ID
    - Output
        - 

## Endpoint

https://ts.glycosmos.org/sparql

## `data`

```sparql
PREFIX xsd: <http://www.w3.org/2001/XMLSchema#>
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX glycan: <http://purl.jp/bio/12/glyco/glycan#>
PREFIX dcterms: <http://purl.org/dc/terms/>
PREFIX sbsmpt: <http://www.glycoinfo.org/glyco/owl/relation#>
PREFIX struct: <https://glytoucan.org/Structures/Glycans/>
PREFIX skos: <http://www.w3.org/2004/02/skos/core#>

SELECT DISTINCT ?parent ?child ?iupac
FROM <http://rdf.glytoucan.org/partner/glycome-db>
FROM <http://rdf.glytoucan.org/partner/bcsdb>
FROM <http://rdf.glytoucan.org/partner/glycoepitope>
FROM <http://rdf.glycosmos.org/glycans/subsumption>
WHERE {
  ?wurcs a ?parent ;
         rdfs:seeAlso ?child ;
         sbsmpt:subsumes* / dcterms:source / glycan:is_from_source / rdfs:seeAlso <http://identifiers.org/taxonomy/9606> .
  OPTIONAL {
    ?wurcs rdfs:label / ^glycan:has_sequence / ^glycan:has_glycosequence / skos:altLabel ?iupac .
  }
  VALUES ?parent {<http://www.glycoinfo.org/glyco/owl/relation#Linkage_defined_saccharide>}
}LIMIT 100
```

## `return`

```javascript
({data}) => {
  const parentIdPrefix = "http://www.glycoinfo.org/glyco/owl/relation#";
  const childIdPrefix = "https://glytoucan.org/Structures/Glycans/";

  const idMap = new Map([["Base_composition_with_linkage", 1],
                         ["Monosaccharide_composition_with_linkage", 2],
                         ["Glycosidic_topology", 3],
                         ["Linkage_defined_saccharide", 4]]);
  let tree = [{
    id: "root",
    label: "root node",
    root: true
  }];
  let chk = {};
  const cap = 40;
  data.results.bindings.forEach(d => {
    let sbsmpt = d.parent.value.replace(parentIdPrefix, "")
    if (!chk[d.parent.value]) {
      chk[d.parent.value] = true;
      tree.push({     
        id: idMap.get(sbsmpt),
        label: sbsmpt.replace(/_/g, " "),
        leaf: false,
        parent: "root"
      });
    }
    let label = "";
    if (d.iupac?.value) {
      label = d.iupac.value;
      if (label.length > cap) {
        label = label.substr(0, cap) + "...";
      }
    } else {
      label = "(" + sbsmpt + ")";
    }

    tree.push({
      id: d.child.value.replace(childIdPrefix, ""),
      label: label,
      leaf: true,
      parent: idMap.get(sbsmpt)
    });
  });
  
  return tree;
}
```