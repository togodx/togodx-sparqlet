#  Taxonomy of human metagenome 16S rRNA analysis from MicrobeDB.jp

## Endpoint

https://microbedb.jp/sparql

## `graph`
```sparql
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX sio: <http://semanticscience.org/resource/>
PREFIX meo: <http://purl.jp/bio/11/meo/>
PREFIX obo: <http://purl.obolibrary.org/obo/>
PREFIX mdbv: <http://purl.jp/bio/11/mdbv#>
PREFIX tax: <http://ddbj.nig.ac.jp/ontologies/taxonomy/>
SELECT DISTINCT ?rank1 ?rank1_label ?rank2 ?rank2_label ?rank3 ?rank3_label
                ?rank4 ?rank4_label ?rank5 ?rank5_label ?rank6 ?rank6_label
                ?rank7 ?rank7_label
WHERE {
  VALUES ?rank { tax:Genus tax:NoRank }
  [] a sio:SIO_001050 ;
     sio:SIO_000008 [
       a mdbv:HostName;
       sio:SIO_000300 "Homo sapiens"
     ] ;
     mdbv:has_analysis [
       a mdbv:TaxonomicAnnotationOfMicrobiomeBasedOn16SrRNA ;
       sio:SIO_000216 [
         mdbv:has_key ?rank1
       ] 
     ] .
  ?rank1 rdfs:label ?rank1_label ;
         tax:rank ?rank .
  OPTIONAL {
    ?rank1 rdfs:subClassOf+ ?rank2.
    ?rank2 tax:rank tax:Family ;
           rdfs:label ?rank2_label .
  }
  OPTIONAL {
    ?rank1 rdfs:subClassOf+ ?rank3 .
    ?rank3 tax:rank tax:Order ;
           rdfs:label ?rank3_label .
  }
  OPTIONAL {
    ?rank1 rdfs:subClassOf+ ?rank4 .
    ?rank4 tax:rank tax:Order ;
           rdfs:label ?rank4_label .
  }
  OPTIONAL {
    ?rank1 rdfs:subClassOf+ ?rank5 .
    ?rank5 tax:rank tax:Phylum ;
            rdfs:label ?rank5_label .
  }
  OPTIONAL {
    ?rank1 rdfs:subClassOf+ ?rank6 .
    ?rank6 tax:rank tax:Kingdom ;
            rdfs:label ?rank6_label .
  }
  OPTIONAL {
    ?rank1 rdfs:subClassOf+ ?rank7 .
    ?rank7 tax:rank tax:Superkingdom ;
           rdfs:label ?rank7_label .
  }
}
```

## `return`
```javascript
({graph}) => {
  const nodePrefix = "http://identifiers.org/taxonomy/";

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
    for (let i = 1; i <= 7; i++) {
      let rank = "rank" + i;
      if (d[rank] && !checked[d[rank].value]) {
        checked[d[rank].value] = true;
        let label = d["rank" + i + "_label"].value;
        let parent = "root";
        let leaf = false;
        for (let j = i + 1; j <= 7; j++) {
          if (d["rank" + j]) {
            parent = d["rank" + j].value.replace(nodePrefix, "");
            break;
          }
        }
        if (i == 1) leaf = true;
        tree.push({
          id: d[rank].value.replace(nodePrefix, ""),
          label: label,
          parent: parent,
          leaf: leaf
        })
      } else {
        break;
      }
    }
  })

  return tree;
}
```