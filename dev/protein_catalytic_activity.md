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

SELECT DISTINCT ?leaf ?label ?parent ?value ?ecclass1 ?ecclass2 ?ecclass3 ?ecclass4 #?ec_sub ?ec_sub2?eccode ?annotation
#SELECT DISTINCT COUNT(?ecclass3) AS ?count ?ecclass3
 FROM <http://rdf.integbio.jp/dataset/togosite/uniprot>
 WHERE {
   ?leaf a up:Protein ;
            up:mnemonic ?label;
            up:proteome ?proteome;
            up:annotation ?annotation .
   ?annotation a up:Catalytic_Activity_Annotation;
            up:catalyticActivity/up:enzymeClass ?eccode .
   BIND(SUBSTR(STR(?eccode),32,99) AS ?value) 
   BIND(SUBSTR(?value, 1, STRLEN(?value)-STRLEN(STRAFTER(STRAFTER(?value ,"."),"."))-1) AS ?ec_sub)
   BIND(SUBSTR(?value, STRLEN(?ec_sub)+2,99) AS ?ec_sub2)
   BIND(xsd:INTEGER(SUBSTR(?value,1,1))*10000000 AS ?ecclass1)
   BIND(xsd:INTEGER(STRAFTER(?ec_sub,"."))*100000 AS ?ecclass2)
   BIND(xsd:INTEGER(STRBEFORE(?ec_sub2,"."))*1000 AS ?ecclass3)
   BIND(xsd:INTEGER(STRAFTER(?ec_sub2,".")) AS ?ecclass4)
   BIND((?ecclass1+?ecclass2+?ecclass3+?ecclass4) AS ?parent)
   FILTER(REGEX(STR(?proteome), "UP000005640"))
}
limit 10
```

## `results`

```javascript
({withAnnotation})=>{
  const idPrefix = "http://purl.uniprot.org/uniprot/";
  let tree = [
    {id: "root", label: "root node", root: true},  
    {id: "10000000", label: "Oxidoreductases", leaf: false, parent: "root"},
    {id: "20000000", label: "Transferases",    leaf: false, parent: "root"},
    {id: "30000000", label: "Hydrolases",      leaf: false, parent: "root"},
    {id: "40000000", label: "Lyases",          leaf: false, parent: "root"},
    {id: "50000000", label: "Isomerases",      leaf: false, parent: "root"},
    {id: "60000000", label: "Ligase",          leaf: false, parent: "root"},
    {id: "70000000", label: "Translocase",     leaf: false, parent: "root"}
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