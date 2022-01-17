# UniProt pathway_Annotation（井手）* 220117 作業中

## Description

- Data sources
    - [UniProt](https://www.uniprot.org/)

- Query
    - Input
        - UniProt ID
    - Output
        - The pathway annotation in Uniprot (unipathway)

## Endpoint
https://integbio.jp/togosite/sparql

## `pathway`
```sparql
PREFIX up: <http://purl.uniprot.org/core/>
PREFIX upid: <http://purl.uniprot.org/uniprot/>
PREFIX taxon: <http://purl.uniprot.org/taxonomy/>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>

SELECT DISTINCT ?uniprot ?mnemonic ?annotation  ?path_1t ?path_1 ?path_2t ?path_2 ?path_3t ?path_3  ?path_4
#SELECT DISTINCT COUNT(?pathway) AS ?count ?pathway ?path
 FROM <http://rdf.integbio.jp/dataset/togosite/uniprot>
 WHERE {
   ?uniprot a up:Protein;
            up:mnemonic ?mnemonic;
            up:annotation ?annotation .
   ?annotation a up:Pathway_Annotation;
               rdfs:seeAlso ?pathway .
   BIND(SUBSTR(STR(?pathway),36,99) AS ?path_1t)  
   BIND(STRBEFORE(?path_1t,".") AS ?path_1) 
   BIND(STRAFTER (?path_1t,".") AS ?path_2t)
   BIND(STRBEFORE(?path_2t,".") AS ?path_2)
   BIND(STRAFTER (?path_2t,".") AS ?path_3t)
   BIND(STRBEFORE(?path_3t,".") AS ?path_3)
   BIND(STRAFTER (?path_3t,".") AS ?path_4)   

   ?uniprot up:proteome ?proteome.
   FILTER(REGEX(STR(?proteome), "UP000005640"))
}
limit 50
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
#limit 10 # offset 8
#}}
#ORDER BY DESC(?count)
```


## `results`

```javascript
({pathway,main})=>{
  const idPrefix = "http://purl.uniprot.org/uniprot/";
  let other_num = 100;  //フロントに表示される数を指定。
  let length = Object.keys(familygen.results.bindings).length;
  let i =1;
  let tree = [
    {
      id: "root",
      label: "root node",
      root: true
    }
  ];
  familygen.results.bindings.map(d => {
    if (i < other_num ){ 
    	tree.push({
      		id: String(i),
      		label: d.family.value,
      		leaf: "false",
      		parent: "root"
    	})
    }else if (i == other_num){
    	tree.push({
      		id: String(other_num),
      		label: "Other",
      		leaf: "false",
      		parent: "root"
    	})
        tree.push({
      		id: String(i+1),
      		label: d.family.value,
      		leaf: "false",
      		parent: String(other_num)
    	})
     }else {
    	tree.push({
      		id: String(i+1),
      		label: d.family.value,
      		leaf: "false",
      		parent: String(other_num)
    	})
     }
     i++;
  });
//  console.log(tree[1].label);
//  console.log(parentgen("G-protein coupled receptor 1 family."));
//  return tree;
  
  main.results.bindings.map(e => {
    tree.push({
        id: e.leaf.value.replace(idPrefix, ""),
        label: e.label.value,
        leaf: "true",
        //parent: e.family.value
        parent: parentgen(e.family.value)
      })
  });
  return tree;
   function parentgen(s){
     let target = tree.filter( f => f["label"] === s)
     return target[0].id ;
   }
}
```