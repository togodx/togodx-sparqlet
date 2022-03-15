#  Taxonomy of metagenome host from MicrobeDB.jp

## Endpoint

https://microbedb.jp/sparql

## `leaf`
```sparql
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX sio: <http://semanticscience.org/resource/>
PREFIX obo: <http://purl.obolibrary.org/obo/>
PREFIX mdbv: <http://purl.jp/bio/11/mdbv#>
PREFIX tax: <http://ddbj.nig.ac.jp/ontologies/taxonomy/>
PREFIX idtax: <http://identifiers.org/taxonomy/>
SELECT DISTINCT ?child ?child_label ?parent
WHERE {
  ?child a sio:SIO_001050 ;
     obo:RO_0002162/rdfs:subClassOf* idtax:408169 ; # metagenome
     sio:SIO_000255/sio:SIO_000255 [
       a mdbv:HostTaxonIDAnnotation ;
       sio:SIO_000671 ?parent
     ] ;
     obo:RO_0002162/rdfs:label ?child_label .
}
```

## `graph`
```sparql
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX sio: <http://semanticscience.org/resource/>
PREFIX obo: <http://purl.obolibrary.org/obo/>
PREFIX mdbv: <http://purl.jp/bio/11/mdbv#>
PREFIX tax: <http://ddbj.nig.ac.jp/ontologies/taxonomy/>
PREFIX idtax: <http://identifiers.org/taxonomy/>
SELECT DISTINCT ?rank1 ?rank1_label ?rank2 ?rank2_label ?rank3 ?rank3_label 
                ?rank4 ?rank4_label ?rank5 ?rank5_label ?rank6 ?rank6_label
                ?rank7 ?rank7_label ?rank8 ?rank8_label
WHERE {
  {
    SELECT DISTINCT ?rank1
    WHERE {
      [] a sio:SIO_001050 ;
         obo:RO_0002162/rdfs:subClassOf* idtax:408169 ; # metagenome
         sio:SIO_000255/sio:SIO_000255 [
           a mdbv:HostTaxonIDAnnotation ;
           sio:SIO_000671 ?rank1
         ] .
    }
  }
  ?rank1 rdfs:label ?rank1_label ;
         tax:rank ?rank .
  OPTIONAL {
    ?rank1 rdfs:subClassOf+ ?rank2.
    ?rank2 tax:rank tax:Genus ;
           rdfs:label ?rank2_label .
  }
  OPTIONAL {
    ?rank1 rdfs:subClassOf+ ?rank3.
    ?rank3 tax:rank tax:Family ;
           rdfs:label ?rank3_label .
  }
  OPTIONAL {
    ?rank1 rdfs:subClassOf+ ?rank4 .
    ?rank4 tax:rank tax:Order ;
           rdfs:label ?rank4_label .
  }
  OPTIONAL {
    ?rank1 rdfs:subClassOf+ ?rank5 .
    ?rank5 tax:rank tax:Class ;
           rdfs:label ?rank5_label .
  }
  OPTIONAL {
    ?rank1 rdfs:subClassOf+ ?rank6 .
    ?rank6 tax:rank tax:Phylum ;
            rdfs:label ?rank6_label .
  }
  OPTIONAL {
    ?rank1 rdfs:subClassOf+ ?rank7 .
    ?rank7 tax:rank tax:Kingdom ;
            rdfs:label ?rank7_label .
  }
  OPTIONAL {
    ?rank1 rdfs:subClassOf+ ?rank8 .
    ?rank8 tax:rank tax:Superkingdom ;
           rdfs:label ?rank8_label .
  }
}
```

## `return`
```javascript
({graph, leaf}) => {
  const nodePrefix = "http://identifiers.org/taxonomy/";
  const idPrefix = "http://identifiers.org/biosample/";

  let tree = [
    {
      id: "root",
      label: "organism",
      root: true
    }
  ];

  let checked = {};
  // 分類の階層
  graph.results.bindings.forEach(d => {
    for (let i = 1; i <= 8; i++) {
      let rank = "rank" + i;
      if (d[rank] && !checked[d[rank].value]) {
        checked[d[rank].value] = true;
        let label = d["rank" + i + "_label"].value;
        let parent = "root";
        for (let j = i + 1; j <= 8; j++) {
          if (d["rank" + j]) {
            parent = d["rank" + j].value.replace(nodePrefix, "");
            break;
          }
        }
        tree.push({
          id: d[rank].value.replace(nodePrefix, ""),
          label: label,
          parent: parent,
        })
      } else if (!d[rank]) {
        continue;
      } else {
        break;
      }
    }
  })
  // アノテーション関係
  leaf.results.bindings.forEach(d => {
    tree.push({
      id: d.child.value.replace(idPrefix, ""),
      label: d.child_label.value,
      leaf: true,
      parent: d.parent.value.replace(nodePrefix, "")
    })
  })
  
  return tree;
}
```