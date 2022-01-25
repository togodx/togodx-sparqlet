# UniProt catalytic_activity（井手）* 220113作業完了

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
PREFIX xsd: <http://www.w3.org/2001/XMLSchema#> 

SELECT DISTINCT ?leaf ?label ?parent #?ecclass ?ecclass1 ?ecclass2 ?ec_sub  #?eccode ?annotation
       #SELECT DISTINCT COUNT(?ec_sub) AS ?count ?ec_sub
 FROM <http://rdf.integbio.jp/dataset/togosite/uniprot>
 WHERE {
   ?leaf a up:Protein ;
            up:mnemonic ?label;
            up:proteome ?proteome;
            up:annotation ?annotation .
   ?annotation a up:Catalytic_Activity_Annotation;
            up:catalyticActivity/up:enzymeClass ?eccode .
   BIND(SUBSTR(STR(?eccode),32,99) AS ?ecclass) 
   BIND(SUBSTR(?ecclass, 1, STRLEN(?ecclass)-STRLEN(STRAFTER(STRAFTER(?ecclass ,"."),"."))-1) AS ?ec_sub)
   BIND(xsd:INTEGER(SUBSTR(?ecclass,1,1))*100 AS ?ecclass1)
   BIND(xsd:INTEGER(STRAFTER(?ec_sub,".")) AS ?ecclass2)
   BIND((?ecclass1+?ecclass2) AS ?parent)
   FILTER(REGEX(STR(?proteome), "UP000005640"))
}
#limit 10
```

## `results`

```javascript
({withAnnotation})=>{
  const idPrefix = "http://purl.uniprot.org/uniprot/";
  let tree = [
    {id: "root", label: "root node", root: true},  
    {id: "01", label: "Oxidoreductases", leaf: false, parent: "root"},
    {id: "02", label: "Transferases",    leaf: false, parent: "root"},
    {id: "03", label: "Hydrolases",      leaf: false, parent: "root"},
    {id: "04", label: "Lyases",          leaf: false, parent: "root"},
    {id: "05", label: "Isomerases",      leaf: false, parent: "root"},
    {id: "06", label: "Ligase",          leaf: false, parent: "root"},
    {id: "07", label: "Translocase",     leaf: false, parent: "root"}
  ];
  
  withAnnotation.results.bindings.map(d => {
    tree.push({
      id: d.leaf.value.replace(idPrefix, ""),
      label: d.label.value,
      leaf: true,
      parent: d.parent.value
    })
  });
  return tree;
}
```