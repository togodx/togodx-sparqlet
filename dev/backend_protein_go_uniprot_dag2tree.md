# UniProt GO classification (守屋)

- backend SPARQLet

## Parameters

* `root` (type: GO) (Req.)
  * default: GO_0005575
  * example: GO_0008150 (biological process), GO_0005575 (cellular component), GO_0003674 (molecular function)

## Endpoint
https://integbio.jp/togosite/sparql

## `graph`
- GO の親子関係
```sparql
PREFIX up: <http://purl.uniprot.org/core/>
PREFIX taxon: <http://purl.uniprot.org/taxonomy/>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX obo: <http://purl.obolibrary.org/obo/>
PREFIX uniprot: <http://purl.uniprot.org/uniprot/>
SELECT DISTINCT ?parent ?child ?parent_label ?child_label
FROM <http://rdf.integbio.jp/dataset/togosite/uniprot>
FROM <http://rdf.integbio.jp/dataset/togosite/go>
WHERE {
  ?child rdfs:subClassOf* obo:{{root}} ;
         rdfs:subClassOf ?parent .
  ?protein up:classifiedWith/rdfs:subClassOf* ?child ;
           up:proteome ?proteome .
  FILTER(REGEX(STR(?child), "GO_"))
  FILTER(REGEX(STR(?parent), "GO_"))
  FILTER(REGEX(STR(?proteome), "UP000005640"))
  ?child rdfs:label ?child_label .
  ?parent rdfs:label ?parent_label .
}
```

## `return`
```javascript
({root, graph}) => {
  const idPrefix = "http://purl.uniprot.org/uniprot/";
  const categoryPrefix = "http://purl.obolibrary.org/obo/";
  const withoutId = "unclassified";
  
  let tree = [
    {
      id: root,
      root: true
    },{
      id: withoutId,
      label: "Unclassified",
      parent: root
    }
  ];
/*
  const extract = (dag, id) => {
    let upper = [];
    let lower = [];
    for (let d of dag) {
      if (d.parents.includes(id)) lower.push(d);
      else upper.push(d);
    }
    return [upper, lower];
  }
  
  const dag2tree = (dag, id) => {
    let [upper, lower] = extract(dag, id);
    console.log(lower);
return upper;
    let index = 1;
    let new_lower = [];
    for (let d of upper) {
      if (d.id == id || d.origin == id) {
        if (d.id == id) d.origin = id;
        d.id = d.id + "_" + index;
        new_lower.concat(lower.map(d => {
          unless (d.origin) d.origin = id;
          d.id = d.id + "_" + index;
          return d;
        }))                            
        index++;
      }
    }
    return upper.concat(new_lower);
  }
*/
  let parents = {};
  for (let d of graph.results.bindings) {
    if (!d.id) console.log(d);
  //  if (!parents[d.id.value]) parents[d.id.value] = [];
  //  parents[d.id.value].push(d.parent.value);
  }

  /*graph.results.bindings = graph.results.bindings.map(d => {
    d.parents = parents[d.id.value];
    return d;
  })*/
console.log(graph.results.bindings);
/*  for (let id of Object.keys(parents)) {
    if (parents[id].length > 1) {
      graph.results.bindings = dag2tree(graph.results.bindings, id);
    }
  }*/
  
  let withAnnotation = {};
  // 親子関係
  graph.results.bindings.map(d => {
    tree.push({
      id: d.child.value.replace(categoryPrefix, ""),
      label: d.child_label.value,
      parent: d.parent.value.replace(categoryPrefix, "")
    })
    if (d.parent.value.replace(categoryPrefix, "") == root && !tree[0].label) tree[0].label = d.parent_label.value; // root の label 挿入
  })

  
  return tree;	
}
```