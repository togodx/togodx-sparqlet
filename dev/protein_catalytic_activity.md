# UniProt catalytic_activity（井手）* 220107-作業中

## Description

- Data sources
    - [UniProt](https://www.uniprot.org/)

- Query
    - Input
        - UniProt ID
    - Output
        - The Catalytic activity based on EC number

## Endpoint
https://integbio.jp/togosite/sparql

## `withAnnotation`
```sparql
PREFIX up: <http://purl.uniprot.org/core/>
PREFIX upid: <http://purl.uniprot.org/uniprot/>
PREFIX taxon: <http://purl.uniprot.org/taxonomy/>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>

SELECT DISTINCT ?uniprot ?eccode_all #?mnemonic ?ecclass_name ?ecclass ?eccode ?annotation  ?comment
#SELECT DISTINCT COUNT(?ecclass) AS ?count ?ecclass 
# FROM <http://rdf.integbio.jp/dataset/togosite/uniprot>
 WHERE {
   #VALUES ?uniprot{ upid:Q96DA6 }
   ?uniprot a up:Protein .
   ?uniprot up:mnemonic ?mnemonic.

   ?uniprot up:annotation ?annotation .
   ?annotation a up:Catalytic_Activity_Annotation .
   ?annotation up:catalyticActivity ?activity .
   ?activity up:enzymeClass ?eccode .
   OPTIONAL{ ?annotation rdfs:comment ?comment .}
  # ?annotation rdfs:comment ?comment .
  # ?annotation 	up:sequence ?isoform .
  BIND(SUBSTR(STR(?eccode),32,1) AS ?ecclass ) 
  BIND(SUBSTR(STR(?eccode),32,100) AS ?eccode_all ) 
  
  BIND(IF(?ecclass = "1","Oxidoreductases" , "") AS ?ecclass_name)
  FILTER(REGEX(STR(?ecclass), "1"))
   
  #{BIND(IF((SUBSTR(?eccode_STR,1,1)) = "2","Transferases" , "") AS ?ecclass)
  #BIND(IF((SUBSTR(?eccode_STR,1,1)) = "3","Hydrolases" , "") AS ?ecclass)
  #BIND(IF((SUBSTR(?eccode_STR,1,1)) = "4","Lyases" , "") AS ?ecclass)
  #BIND(IF((SUBSTR(?eccode_STR,1,1)) = "5","Isomerases" , "") AS ?ecclass)
  #BIND(IF((SUBSTR(?eccode_STR,1,1)) = "6","Ligases" , "") AS ?ecclass)
  #BIND(IF((SUBSTR(?eccode_STR,1,1)) = "7","Translocases" , "") AS ?ecclass)
  
  #FILTER(REGEX(STR(?proteome), "UP000005640"))
  
  ?uniprot up:proteome ?proteome.
  FILTER(REGEX(STR(?proteome), "UP000005640"))
}
Order by ?eccode_all
limit 10
```

## `withoutAnnotation`
```sparql
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX taxon: <http://purl.uniprot.org/taxonomy/>
PREFIX core: <http://purl.uniprot.org/core/>
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
      binId: Number(d.value.value) + 1,
      binLabel: d.value.value
    }
  });
}
```