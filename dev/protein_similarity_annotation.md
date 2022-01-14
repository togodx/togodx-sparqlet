# UniProt similarity_annotation（井手）* 220114作業中

## Description

- Data sources
    - [UniProt](https://www.uniprot.org/)

- Query
    - Input
        - UniProt ID
    - Output
        - The similarity (family) annotation in Uniprot

## Endpoint
https://integbio.jp/togosite/sparql

## `familygen`
```sparql
PREFIX up: <http://purl.uniprot.org/core/>
PREFIX upid: <http://purl.uniprot.org/uniprot/>
PREFIX taxon: <http://purl.uniprot.org/taxonomy/>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>

SELECT ?family
WHERE{ 
  FILTER(?count>1) 
  {
  SELECT DISTINCT COUNT(?family) AS ?count ?family ?length#?uniprot  
  #SELECT DISTINCT ?leaf ?label ?family #?uniprot  
  FROM <http://rdf.integbio.jp/dataset/togosite/uniprot>
  WHERE {
     ?leaf a up:Protein;
            up:mnemonic ?label;
            up:reviewed true;        #reviewのみに絞った。
            up:proteome ?proteome;
            up:annotation ?annotation .
     ?annotation a up:Similarity_Annotation;
                 rdfs:comment ?comment .
     BIND(SUBSTR(?comment,16,150)AS ?family )
     FILTER(REGEX(STR(?proteome), "UP000005640"))
  }
}}
ORDER BY DESC(?count)
limit 100
```

## `main`
```sparql
PREFIX up: <http://purl.uniprot.org/core/>
PREFIX upid: <http://purl.uniprot.org/uniprot/>
PREFIX taxon: <http://purl.uniprot.org/taxonomy/>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>

#SELECT ?family
#WHERE{ 
#  FILTER(?count>1) 
#  {
#  SELECT DISTINCT COUNT(?family) AS ?count ?family ?length#?uniprot  
SELECT DISTINCT ?leaf ?label ?family #?uniprot  
  FROM <http://rdf.integbio.jp/dataset/togosite/uniprot>
  WHERE {
     ?leaf a up:Protein;
            up:mnemonic ?label;
            up:reviewed true;        #reviewのみに絞った。
            up:proteome ?proteome;
            up:annotation ?annotation .
     ?annotation a up:Similarity_Annotation;
                 rdfs:comment ?comment .
     BIND(SUBSTR(?comment,16,150)AS ?family )
     FILTER(REGEX(STR(?proteome), "UP000005640"))
  }
limit 100
#}}
#ORDER BY DESC(?count)
```


## `results`

```javascript
({familygen,main})=>{
  const idPrefix = "http://purl.uniprot.org/uniprot/";
  let other_num = 50;  //フロントに表示される数は50にする。
  let length = Object.keys(familygen.results.bindings).length;    //length of rfree
  let i =1;
  let tree = [
    {
      id: "root",
      label: "root node",
      root: true
    }
  ];
  familygen.results.bindings.map(d => {
    tree.push({
      id: "i",
      label: d.family.value,
      leaf: "false",
      parent: "root"
    })
    i++;
  });
  return tree;
  
  
//  main.results.bindings.map(e => {
//    tree.push({
//      id: e.leaf.value.replace(idPrefix, ""),
//      label: e.label.value,
//      leaf: true,
//      parent: e.parent.value
//    })
//  });
//  return tree;
}
```