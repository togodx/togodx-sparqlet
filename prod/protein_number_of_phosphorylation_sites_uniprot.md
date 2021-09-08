# UniProt phosphorylation-site count（守屋）

## Description

- Data sources
    - [UniProt](https://www.uniprot.org/)

- Query
    - Input
        - UniProt ID
    - Output
        - The number of phosphorylation site

## Endpoint
https://integbio.jp/togosite/sparql

## `withAnnotation`
- 
```sparql
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX taxon: <http://purl.uniprot.org/taxonomy/>
PREFIX core: <http://purl.uniprot.org/core/>
PREFIX up: <http://purl.uniprot.org/uniprot/>
PREFIX xsd: <http://www.w3.org/2001/XMLSchema#>
SELECT DISTINCT ?leaf ?label (COUNT (DISTINCT ?annotation) AS ?value)
FROM <http://rdf.integbio.jp/dataset/togosite/uniprot>
WHERE
{
  ?leaf a core:Protein ;
         core:mnemonic ?label ;
         core:organism taxon:9606 ;
         core:proteome ?proteome ;
         core:annotation ?annotation .
  ?annotation a core:Modified_Residue_Annotation ;
              rdfs:comment ?mod .
  FILTER(REGEX(STR(?proteome), "UP000005640"))
  FILTER (REGEX (?mod, "Phosphoserin") || REGEX (?mod, "Phosphothreonine") || REGEX (?mod, "Phosphotyrosine"))  # "Phospho[STY]; by hoge"
}
```

## `withoutAnnotation`
```sparql
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX taxon: <http://purl.uniprot.org/taxonomy/>
PREFIX core: <http://purl.uniprot.org/core/>
PREFIX up: <http://purl.uniprot.org/uniprot/>
PREFIX xsd: <http://www.w3.org/2001/XMLSchema#>
SELECT DISTINCT ?leaf ?label ?value
FROM <http://rdf.integbio.jp/dataset/togosite/uniprot>
WHERE {
  ?leaf a core:Protein ;
           core:mnemonic ?label ;
           core:organism taxon:9606 ;
           core:proteome ?proteome .
  FILTER(REGEX(STR(?proteome), "UP000005640"))
  MINUS {
    ?leaf core:annotation ?annotation .
    ?annotation a core:Modified_Residue_Annotation ;
                rdfs:comment ?mod .
    FILTER (REGEX (?mod, "Phosphoserin") || REGEX (?mod, "Phosphothreonine") || REGEX (?mod, "Phosphotyrosine"))  # "Phospho[STY]; by hoge"
  }
  BIND ("0" AS ?value)
}
```

## `results`

```javascript
({withAnnotation, withoutAnnotation})=>{
  const idPrefix = "http://purl.uniprot.org/uniprot/";
  
  return withAnnotation.results.bindings.concat(withoutAnnotation.results.bindings).map(d => {
    return {
      id: d.leaf.value.replace(idPrefix, ""),
      label: d.label.value,
      value: Number(d.value.value),
      binBegin: Number(d.value.value),
      binEnd: Number(d.value.value),
      binLabel: d.value.value
    }
  });
}
```