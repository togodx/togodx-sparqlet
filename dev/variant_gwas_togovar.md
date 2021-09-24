# variant GWAS (Genome-wide association study) (三橋)

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
SELECT DISTINCT ?child ?parent_label ?parent (REPLACE (STR(?dbsnp), "http://identifiers.org/dbsnp/", "") AS ?child_label)
FROM <http://rdf.integbio.jp/dataset/togosite/variation>
FROM <http://rdf.integbio.jp/dataset/togosite/gwas-catalog>
FROM <http://rdf.integbio.jp/dataset/togosite/efo>
WHERE {
  GRAPH <http://rdf.integbio.jp/dataset/togosite/efo>{
    ?parent rdfs:label ?parent_label ;
            a owl:Class .
  }
  GRAPH <http://rdf.integbio.jp/dataset/togosite/gwas-catalog>{
    ?parent ^terms:mapped_trait_uri/terms:dbsnp_url ?dbsnp .
  }
  GRAPH <http://rdf.integbio.jp/dataset/togosite/variation>{
    ?dbsnp ^rdfs:seeAlso/dct:identifier ?child .
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
SELECT DISTINCT ?child ?parent ?child_label ?parent_label
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
  ?trait ^terms:mapped_trait_uri/terms:dbsnp_url/^rdfs:seeAlso ?togovar.
}
```

## `return`
```javascript
({leaf, graph}) => {
  
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
      id: d.child.value.replace(idPrefix, ""),
      label: d.child_label.value,
      leaf: true,
      parent: d.parent.value.split(/\//).slice(-1)[0]
    })
  })
  
  return tree;	
}
```