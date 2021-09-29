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

## `parents_graph`
- GO の親全部
```sparql
PREFIX up: <http://purl.uniprot.org/core/>
PREFIX taxon: <http://purl.uniprot.org/taxonomy/>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX obo: <http://purl.obolibrary.org/obo/>
PREFIX uniprot: <http://purl.uniprot.org/uniprot/>
SELECT DISTINCT ?parent ?child
FROM <http://rdf.integbio.jp/dataset/togosite/uniprot>
FROM <http://rdf.integbio.jp/dataset/togosite/go>
WHERE {
  ?child rdfs:subClassOf* obo:{{root}} ;
         rdfs:subClassOf* ?parent .
  FILTER(REGEX(STR(?child), "GO_"))
  FILTER(REGEX(STR(?parent), "GO_"))
}
```

## `return`
```javascript
({root, graph, parents_graph}) => {
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
    let index = 1;
    let new_lower = [];
    for (let d of upper) {
      if (d.id == id || (d.origin && d.origin == id)) {
        if (d.id == id) d.origin = id;
        d.id = d.id + "_" + index;
        new_lower = new_lower.concat(lower.map(d => {
          if (!d.origin) d.origin = id;
          d.id = d.id + "_" + index;
          return d;
        }))                            
        index++;
      }
    }
    return upper.concat(new_lower);
  }

  let parents = {};
  parents_graph.results.bindings.map(d => {
    const id = d.child.value.replace(categoryPrefix, "");
    if (!parents[id]) parents[id] = [];
    parents[id].push(d.parent.value.replace(categoryPrefix, ""));
  })
  
  let dag = graph.results.bindings.map(d => {
    if (d.parent.value.replace(categoryPrefix, "") == root && !tree[0].label) tree[0].label = d.parent_label.value; // root の label 挿入
    return {
      id: d.child.value.replace(categoryPrefix, ""),
      label: d.child_label.value,
      parent: d.parent.value.replace(categoryPrefix, ""),
      parents: parents[d.child.value.replace(categoryPrefix, "")]
    }
  })

  for (let id of Object.keys(parents)) {
    if (parents[id].length > 1) {
      dag = dag2tree(dag, id);
      break;
    }
  }

  return tree.concat(dag);	
}
```