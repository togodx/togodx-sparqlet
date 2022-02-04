# UniProt pathway_annotation（井手）* 220204完成

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

## `getpathway1`
```sparql
PREFIX upa: <http://purl.uniprot.org/unipathway/>
PREFIX up: <http://purl.uniprot.org/core/>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX skos:<http://www.w3.org/2004/02/skos/core#>
SELECT DISTINCT ?parent ?parent_label 
 FROM <http://rdf.integbio.jp/dataset/togosite/uniprot/pathways>
WHERE {
  ?parent rdf:type up:Pathway;
          rdfs:label ?parent_label;  
          rdfs:subClassOf ?root;
          skos:narrower ?child .     
  Filter(REGEX(?root,up:Pathway))
}
ORDER by ?parent
#limit 10
```

## `getpathway2`
```sparql
PREFIX upa: <http://purl.uniprot.org/unipathway/>
PREFIX up: <http://purl.uniprot.org/core/>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX skos:<http://www.w3.org/2004/02/skos/core#>
SELECT DISTINCT ?parent ?parent_label ?child  ?child_label 
 FROM <http://rdf.integbio.jp/dataset/togosite/uniprot/pathways>
WHERE {
  ?parent rdf:type up:Pathway;
          rdfs:label ?parent_label;  
          skos:narrower ?child .    
  ?child  rdfs:label ?child_label_temp .
  BIND((SUBSTR((STRAFTER(STR(?child_label_temp),STR(?parent_label))),3,99))AS ?child_label)
}
ORDER by ?parent
#limit 10
```

## `main`
```sparql
PREFIX up: <http://purl.uniprot.org/core/>
PREFIX upid: <http://purl.uniprot.org/uniprot/>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX upa: <http://purl.uniprot.org/unipathway/>

SELECT DISTINCT ?leaf ?label ?parent  
 FROM <http://rdf.integbio.jp/dataset/togosite/uniprot>
WHERE {
   ?leaf a up:Protein;
            up:mnemonic ?label;
            up:annotation ?annotation;
            up:proteome ?proteome.
   ?annotation a up:Pathway_Annotation;
               rdfs:seeAlso ?parent .
   FILTER(REGEX(STR(?proteome), "UP000005640"))
}
#limit 10
```

## `results`

```javascript
({getpathway1, getpathway2, main})=>{
  const idPrefix_unipathway= "http://purl.uniprot.org/unipathway/";
  const idPrefix_uniprot="http://purl.uniprot.org/uniprot/";
  let tree = [
    {
      id: "root",
      label: "root node",
      root: true
    }
  ];
  getpathway1.results.bindings.map(d => {
    tree.push({
        id: d.parent.value.replace(idPrefix_unipathway, ""),
        label: d.parent_label.value,
        leaf: "false",
        parent: "root"
      })
  });
  getpathway2.results.bindings.map(d => {
    tree.push({
        id: d.child.value.replace(idPrefix_unipathway, ""),
        label: d.child_label.value,
        leaf: "false",
        parent: d.parent.value.replace(idPrefix_unipathway, "")
      })
  });
  main.results.bindings.map(d => {
    tree.push({
        id: d.leaf.value.replace(idPrefix_uniprot, ""),
        label: d.label.value,
        leaf: "true",
        parent: d.parent.value.replace(idPrefix_unipathway, "")
      })
  });
  return tree;
}
```