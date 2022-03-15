#  Taxonomy of microbes in human from MicrobeDB.jp

## Endpoint

https://microbedb.jp/sparql

## `graph`
```sparql
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX sio: <http://semanticscience.org/resource/>
PREFIX mdbv: <http://purl.jp/bio/11/mdbv#>
PREFIX obo: <http://purl.obolibrary.org/obo/>
PREFIX tax: <http://ddbj.nig.ac.jp/ontologies/taxonomy/>
PREFIX idtax: <http://identifiers.org/taxonomy/>
SELECT DISTINCT ?rank0 ?rank0_label ?rank1 ?rank1_label ?rank2 ?rank2_label
                ?rank3 ?rank3_label ?rank4 ?rank4_label ?rank5 ?rank5_label
                ?rank6 ?rank6_label ?rank7 ?rank7_label
WHERE {
  [] a sio:SIO_001050 ;
     sio:SIO_000255/sio:SIO_000255 [
       a mdbv:HostTaxonIDAnnotation ;
       sio:SIO_000671 idtax:9606
     ] ;
     obo:RO_0002162 ?rank0 .
  ?rank0 rdfs:label ?rank0_label_pre .
  OPTIONAL {
    ?rank0 rdfs:subClassOf+ ?rank1 .
    ?rank1 tax:rank tax:Genus ;
           rdfs:label ?rank1_label .
  }
  OPTIONAL {
    ?rank0 rdfs:subClassOf+ ?rank2.
    ?rank2 tax:rank tax:Family ;
           rdfs:label ?rank2_label .
  }
  OPTIONAL {
    ?rank0 rdfs:subClassOf+ ?rank3 .
    ?rank3 tax:rank tax:Order ;
           rdfs:label ?rank3_label .
  }
  OPTIONAL {
    ?rank0 rdfs:subClassOf+ ?rank4 .
    ?rank4 tax:rank tax:Class ;
           rdfs:label ?rank4_label .
  }
  OPTIONAL {
    ?rank0 rdfs:subClassOf+ ?rank5 .
    ?rank5 tax:rank tax:Phylum ;
            rdfs:label ?rank5_label .
  }
  OPTIONAL {
    ?rank0 rdfs:subClassOf+ ?rank6 .
    ?rank6 tax:rank tax:Kingdom ;
            rdfs:label ?rank6_label .
  }
  OPTIONAL {
    ?rank0 rdfs:subClassOf+ ?rank7 .
    ?rank7 tax:rank tax:Superkingdom ;
           rdfs:label ?rank7_label .
  }
  MINUS { ?rank0 rdfs:subClassOf* idtax:408169 } # w/o metagenome
  BIND(STR(?rank0_label_pre) AS ?rank0_label) # uniform with/without xsd:string datatype
}
```

## `return`
```javascript
({graph}) => {
  const idPrefix = "http://identifiers.org/taxonomy/";

  let tree = [
    {
      id: "root",
      label: "organism",
      root: true
    }
  ];

  let checked = {};
  graph.results.bindings.forEach(d => {
    for (let i = 0; i <= 7; i++) {
      let rank = "rank" + i;
      if (d[rank] && !checked[d[rank].value]) {
        checked[d[rank].value] = true;
        let label = d["rank" + i + "_label"].value;
        let parent = "root";
        let leaf = false;
        for (let j = i + 1; j <= 7; j++) {
          if (d["rank" + j]) {
            parent = d["rank" + j].value.replace(idPrefix, "");
            break;
          }
        }
        if (i == 0) leaf = true;
        tree.push({
          id: d[rank].value.replace(idPrefix, ""),
          label: label,
          parent: parent,
          leaf: leaf
        })
      }  else if (!d[rank]) {
        continue;
      } else {
        break;
      }
    }
  })

  return tree;
}
```