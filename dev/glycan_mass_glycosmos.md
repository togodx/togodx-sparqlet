# Glycan mass （山本・池田）

## Description

- Data sources
    - [GlyCosmos](https://glycosmos.org/data)

- Query
    - Input
        - GlyTouCan ID
    - Output
        - Mass range

## Endpoint

https://ts.glycosmos.org/sparql

## `data`

```sparql
PREFIX xsd: <http://www.w3.org/2001/XMLSchema#>
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX glycan: <http://purl.jp/bio/12/glyco/glycan#>
PREFIX dcterms: <http://purl.org/dc/terms/>
PREFIX mass: <https://glycoinfo.gitlab.io/wurcsframework/org/glycoinfo/wurcsframework/1.0.1/wurcsframework-1.0.1.jar#>
PREFIX sbsmpt: <http://www.glycoinfo.org/glyco/owl/relation#>
PREFIX glyconavi: <http://glyconavi.org/owl#>
PREFIX struct: <https://glytoucan.org/Structures/Glycans/>
PREFIX skos: <http://www.w3.org/2004/02/skos/core#>

SELECT DISTINCT ?mass ?glytoucan ?sbsmpt ?iupac
WHERE {
  {
    ?s mass:WURCSMassCalculator ?mass ;
       rdfs:seeAlso ?glytoucan ;
       a ?sbsmpt ;
       sbsmpt:subsumes* / dcterms:source / glycan:is_from_source / rdfs:seeAlso <http://identifiers.org/taxonomy/9606> .
    OPTIONAL {
     ?s rdfs:label / ^glycan:has_sequence / ^glycan:has_glycosequence / skos:altLabel ?iupac .
    }
  } UNION {
    ?s mass:WURCSMassCalculator ?mass ;
       rdfs:seeAlso ?glytoucan ;
       a ?sbsmpt .
    ?s sbsmpt:subsumes* / rdfs:label / ^glycan:has_sequence / ^glycan:has_glycosequence ?gtc .
    ?gtc ^glycan:has_glycan / glycan:has_taxon <http://rdf.glycoinfo.org/source/9606> .
    OPTIONAL {
      ?gtc skos:altLabel ?iupac .
    }
  }
}
```

## `return`
```javascript
({data}) => {
  const idPrefix = "https://glytoucan.org/Structures/Glycans/";
  
  let tree = [];
  data.results.bindings.forEach(d => {
    const bin = 200;
    const num = parseInt(Number(d.mass.value) / bin) * bin;
    const binLabel = num + "-" + (num + bin) + " Da";
    const binId = num / bin;
    const cap = 40;
    let label = "";
    if (d.iupac?.value) {
      label = d.iupac.value;
      if (label.length > cap) {
        label = label.substr(0, cap) + "...";
      }
    } else if (d.sbsmpt?.value) {
      label = "(" + d.sbsmpt.value.replace("http://www.glycoinfo.org/glyco/owl/relation#", "").replace(/_/g, " ") + ")";
    }
    tree.push({
      id: d.glytoucan.value.replace(idPrefix, ""),
      label: label,
      value: Number(d.mass.value),
      binId: binId,
      binLabel: binLabel
    });
  });
  
  return tree;
}
```
