# UniProt pathway_annotation（井手）* 220117 作業中

## Description

- Data sources
    - [UniProt](https://www.uniprot.org/)

- Query
    - Input
        - UniProt ID
    - Output
        - The pathway annotation in Uniprot (unipathway)

## Endpoint
https://integbio.jp/togosite_dev/sparql

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
PREFIX obo: <http://purl.obolibrary.org/obo/>
PREFIX up: <http://purl.uniprot.org/core/>
PREFIX upid: <http://purl.uniprot.org/uniprot/>
PREFIX taxon: <http://purl.uniprot.org/taxonomy/>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX oboin: <http://www.geneontology.org/formats/oboInOwl#>

SELECT DISTINCT ?s ?label ?o2
#FROM <http://graph/unipathway>
WHERE {
  #VALUES ?s { obo:UPa_UPA00026 }
  #VALUES ?o2 { "pathway" }
  ?s ?p ?o .
  ?s rdfs:label ?label .
  ?s oboin:hasOBONamespace ?o2 .
  Filter(REGEX(STR(?o2),"pathway"))
} 
LIMIT 1000
```


## `results`

```javascript
({pathway})=>{
  const idPrefix = "http://purl.uniprot.org/uniprot/";
  let other_num = 100;  //フロントに表示される数を指定。
//  let length = Object.keys(familygen.results.bindings).length;
  let i =1;
  let tree = [
    {
      id: "root",
      label: "root node",
      root: true
    }
  ];
  var x = new XMLHttpRequest();

  x.onreadystatechange = function () {
  if (x.readyState == 4 && x.status == 200)
      {
       console.log("log1");
       console.log(x.responseText); // var doc = x.responseXML;
       // …
      }
  };
  x.open("GET", "https://raw.githubusercontent.com/geneontology/unipathway/master/upa.owl", true);
  x.send();
  console.log("log2");

//  console.log(doc);
//  console.log(parentgen("G-protein coupled receptor 1 family."));
//  return tree;
  
//  main.results.bindings.map(e => {
//    tree.push({
//        id: e.leaf.value.replace(idPrefix, ""),
//        label: e.label.value,
//        leaf: "true",
        //parent: e.family.value
//       parent: parentgen(e.family.value)
//      })
//  });
//  return tree;
//   function parentgen(s){
//     let target = tree.filter( f => f["label"] === s)
//     return target[0].id ;
//   }
}
```