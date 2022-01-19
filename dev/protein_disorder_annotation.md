# UniProt disorder_annotation（井手）* 220119作業中

## Description

- Data sources
    - [UniProt](https://www.uniprot.org/)

- Query
    - Input
        - UniProt ID
    - Output
        - The disorder annotation in Region_Annotation of Uniprot

## Endpoint
https://integbio.jp/togosite/sparql

## `disorder`
```sparql
PREFIX up: <http://purl.uniprot.org/core/>
PREFIX upid: <http://purl.uniprot.org/uniprot/>
PREFIX taxon: <http://purl.uniprot.org/taxonomy/>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX xsd: <http://www.w3.org/2001/XMLSchema#> 
PREFIX faldo: <http://biohackathon.org/resource/faldo#>

#SELECT DISTINCT COUNT (?length) AS ?count ?length  
SELECT DISTINCT ?uniprot ?mnemonic ?annotation ?length # ?begin_position ?end_position
 FROM <http://rdf.integbio.jp/dataset/togosite/uniprot>
 WHERE {
   ?uniprot a up:Protein ;
            up:mnemonic ?mnemonic;
            up:annotation ?annotation .
   ?annotation a up:Region_Annotation;
               rdfs:comment "Disordered";
               up:range ?range .
   ?range rdf:type faldo:Region;
          faldo:begin/faldo:position ?begin_position;
          faldo:end/faldo:position ?end_position .
   BIND ((?end_position-?begin_position) AS ?length) .   
 
   #?begin faldo:reference ?begin_reference_seq .
   #?begin_reference_seq rdf:value ?value .
   #BIND ((substr(str(?value),?begin_position,?end_position-?begin_position)) AS ?seq) .
   
   ?uniprot up:proteome ?proteome.
   FILTER(REGEX(STR(?proteome), "UP000005640"))
}
Order by (?length)
limit 100
```

## `binIDgen`
```sparql
PREFIX up: <http://purl.uniprot.org/core/>
PREFIX upid: <http://purl.uniprot.org/uniprot/>
PREFIX taxon: <http://purl.uniprot.org/taxonomy/>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX xsd: <http://www.w3.org/2001/XMLSchema#> 
PREFIX faldo: <http://biohackathon.org/resource/faldo#>

SELECT DISTINCT ?length 
 FROM <http://rdf.integbio.jp/dataset/togosite/uniprot>
 WHERE {
   ?uniprot a up:Protein ;
            up:mnemonic ?mnemonic;
            up:annotation ?annotation .
   ?annotation a up:Region_Annotation;
               rdfs:comment "Disordered";
               up:range ?range .
   ?range rdf:type faldo:Region;
          faldo:begin/faldo:position ?begin_position;
          faldo:end/faldo:position ?end_position .
   BIND ((?end_position-?begin_position) AS ?length) .   
   ?uniprot up:proteome ?proteome.
   FILTER(REGEX(STR(?proteome), "UP000005640"))
}
Order by (?length)
#limit 100
```


## `results`

```javascript
({familygen,main})=>{
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