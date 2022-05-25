# variant GWAS (Genome-wide association study) (三橋, 守屋)

## Description

- Data sources
    -  [TogoVar](https://togovar.biosciencedbc.jp/?) (limited to variants with frequency data in Japanese populations)
- Query
    - Input
        - TogoVar id
    - Output
        -  [Mapped trait used in GWAS Catalog](https://www.ebi.ac.uk/gwas/docs/ontology)

## Endpoint

https://integbio.jp/togosite/sparql

## `leaf`
```sparql
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX owl: <http://www.w3.org/2002/07/owl#>
PREFIX dct: <http://purl.org/dc/terms/>
PREFIX terms: <http://med2rdf.org/gwascatalog/terms/>
SELECT DISTINCT ?parent ?child ?parent_label ?child_label
FROM <http://rdf.integbio.jp/dataset/togosite/variation>
FROM <http://rdf.integbio.jp/dataset/togosite/gwas-catalog>
FROM <http://rdf.integbio.jp/dataset/togosite/efo>
WHERE {
  GRAPH <http://rdf.integbio.jp/dataset/togosite/efo>{
    ?parent rdfs:label ?parent_label ;
            a owl:Class .
  }
  GRAPH <http://rdf.integbio.jp/dataset/togosite/gwas-catalog>{
    ?parent ^terms:mapped_trait_uri/rdfs:seeAlso ?child_label .
  }
  GRAPH <http://rdf.integbio.jp/dataset/togosite/variation>{
    ?child_label ^rdfs:seeAlso/dct:identifier ?child .
  }
}
```

## `graph`
```sparql
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX owl: <http://www.w3.org/2002/07/owl#>
PREFIX dct: <http://purl.org/dc/terms/>
PREFIX terms: <http://med2rdf.org/gwascatalog/terms/>
PREFIX efo: <http://www.ebi.ac.uk/efo/>
SELECT DISTINCT ?parent ?child ?parent_label ?child_label
FROM <http://rdf.integbio.jp/dataset/togosite/variation>
FROM <http://rdf.integbio.jp/dataset/togosite/gwas-catalog>
FROM <http://rdf.integbio.jp/dataset/togosite/efo>
WHERE {
  VALUES ?root {  efo:EFO_0000001  } 
  GRAPH <http://rdf.integbio.jp/dataset/togosite/efo> {
    ?child a owl:Class ;
           rdfs:subClassOf* ?root ;
           rdfs:subClassOf ?parent ;
           rdfs:label ?child_label .
    ?parent a owl:Class ;
            rdfs:label ?parent_label .
    ?trait rdfs:subClassOf* ?child .
  }
  ?trait ^terms:mapped_trait_uri/rdfs:seeAlso/^rdfs:seeAlso ?togovar.
}
```

## `return`
```javascript
({leaf, graph}) => {
  const childLabelPrefix = "http://identifiers.org/dbsnp/";
  
  let tree = [
    {
      id: "EFO_0000001",
      label: "experimental factor",
      root: true
    }
  ];

  // 親子関係
  graph.results.bindings.map(d => {
    tree.push({
      id: d.child.value.split(/\//).slice(-1)[0],
      label: d.child_label.value,
      parent: d.parent.value.split(/\//).slice(-1)[0]
    })
  })
  // アノテーション関係
  leaf.results.bindings.map(d => {
    tree.push({
      id: d.child.value,
      label: d.child_label.value.replace(childLabelPrefix, ""),
      leaf: true,
      parent: d.parent.value.split(/\//).slice(-1)[0]
    })
  })
  
  return tree;	
}
```