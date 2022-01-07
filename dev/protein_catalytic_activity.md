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

SELECT DISTINCT ?leaf ?value ?label ?bin #?mnemonic ?ecclass_name ?ecclass ?eccode ?annotation  ?comment
#SELECT DISTINCT COUNT(?ecclass) AS ?count ?ecclass 
 FROM <http://rdf.integbio.jp/dataset/togosite/uniprot>
 WHERE {
   #VALUES ?leaf{ upid:Q96DA6 }
   ?leaf a up:Protein .
   ?leaf up:mnemonic ?mnemonic.

   ?leaf up:annotation ?annotation .
   ?annotation a up:Catalytic_Activity_Annotation .
   ?annotation up:catalyticActivity ?activity .
   ?activity up:enzymeClass ?eccode .
  # OPTIONAL{ ?annotation rdfs:comment ?comment .}
  # ?annotation rdfs:comment ?comment .
  # ?annotation 	up:sequence ?isoform .
  BIND(SUBSTR(STR(?eccode),32,1) AS ?bin ) 
  BIND(SUBSTR(STR(?eccode),32,100) AS ?value ) 
  
  BIND(IF(?bin = "1","Oxidoreductases" , "") AS ?label)
  FILTER(REGEX(STR(?bin), "1"))
   
  #{BIND(IF((SUBSTR(?eccode_STR,1,1)) = "2","Transferases" , "") AS ?ecclass)
  #BIND(IF((SUBSTR(?eccode_STR,1,1)) = "3","Hydrolases" , "") AS ?ecclass)
  #BIND(IF((SUBSTR(?eccode_STR,1,1)) = "4","Lyases" , "") AS ?ecclass)
  #BIND(IF((SUBSTR(?eccode_STR,1,1)) = "5","Isomerases" , "") AS ?ecclass)
  #BIND(IF((SUBSTR(?eccode_STR,1,1)) = "6","Ligases" , "") AS ?ecclass)
  #BIND(IF((SUBSTR(?eccode_STR,1,1)) = "7","Translocases" , "") AS ?ecclass)
  
  #FILTER(REGEX(STR(?proteome), "UP000005640"))
  
  ?leaf up:proteome ?proteome.
  FILTER(REGEX(STR(?proteome), "UP000005640"))
}
Order by ?value
limit 10
```

## `results`

```javascript
({withAnnotation})=>{
  const idPrefix = "http://purl.uniprot.org/uniprot/";
  
  return withAnnotation.results.bindings.map(d => {
    return {
      id: d.leaf.value.replace(idPrefix, ""),
      label: d.label.value,
      value: Number(d.value.value),
      binId: Number(d.bin.value) + 1,
      binLabel: d.bin.value
    }
  });
}
```