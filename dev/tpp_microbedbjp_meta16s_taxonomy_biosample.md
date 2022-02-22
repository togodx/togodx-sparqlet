#  Taxonomy of metagenome 16S rRNA analysis from MicrobeDB.jp

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
SELECT DISTINCT ?taxon ?taxon_label ?rank1 ?rank1_label ?rank2 ?rank2_label
                ?rank3 ?rank3_label ?rank4 ?rank4_label ?rank5 ?rank5_label 
                ?rank6 ?rank6_label ?rank7 ?rank7_label
WHERE {
  [] a sio:SIO_001050 ;
     mdbv:has_analysis [
       a mdbv:TaxonomicAnnotationOfMicrobiomeBasedOn16SrRNA ;
       sio:SIO_000216 [
         mdbv:has_key ?taxon
       ] 
     ] .
  ?taxon rdfs:label ?taxon_label .
  OPTIONAL {
    ?taxon rdfs:subClassOf+ ?rank1 .
    ?rank1 tax:rank tax:Genus ;
           rdfs:label ?rank1_label .
  }
  OPTIONAL {
    ?taxon rdfs:subClassOf+ ?rank2.
    ?rank2 tax:rank tax:Family ;
           rdfs:label ?rank2_label .
  }
  OPTIONAL {
    ?taxon rdfs:subClassOf+ ?rank3 .
    ?rank3 tax:rank tax:Order ;
           rdfs:label ?rank3_label .
  }
  OPTIONAL {
    ?taxon rdfs:subClassOf+ ?rank4 .
    ?rank4 tax:rank tax:Order ;
           rdfs:label ?rank4_label .
  }
  OPTIONAL {
    ?taxon rdfs:subClassOf+ ?rank5 .
    ?rank5 tax:rank tax:Phylum ;
            rdfs:label ?rank5_label .
  }
  OPTIONAL {
    ?taxon rdfs:subClassOf+ ?rank6 .
    ?rank6 tax:rank tax:Kingdom ;
            rdfs:label ?rank6_label .
  }
  OPTIONAL {
    ?taxon rdfs:subClassOf+ ?rank7 .
    ?rank7 tax:rank tax:Superkingdom ;
           rdfs:label ?rank7_label .
  }
}
```

## `leaf`
```sparql
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX sio: <http://semanticscience.org/resource/>
PREFIX meo: <http://purl.jp/bio/11/meo/>
PREFIX obo: <http://purl.obolibrary.org/obo/>
PREFIX mdbv: <http://purl.jp/bio/11/mdbv#>
PREFIX tax: <http://ddbj.nig.ac.jp/ontologies/taxonomy/>
SELECT DISTINCT ?sample ?sample_label ?taxon
WHERE {
  ?sample a sio:SIO_001050 ;
          rdfs:label ?sample_label ;
          mdbv:has_analysis [
            a mdbv:TaxonomicAnnotationOfMicrobiomeBasedOn16SrRNA ;
            sio:SIO_000216 [
              mdbv:has_key ?taxon
            ] 
          ] .
}
```


## `return`
```javascript
({graph, leaf}) => {
  const idPrefix = "http://identifiers.org/biosample/";
  const nodePrefix = "http://identifiers.org/taxonomy/";

  let tree = [
    {
      id: "root",
      label: "organism",
      root: true
    }
  ];

  let checked = {};
  graph.results.bindings.forEach(d => {
    if (!checked[d.taxon.value]) {
      checked[d.taxon.value] = true;
      let parent = "root";
      if (d.rank1) parent = d.rank1.value.replace(nodePrefix, "");
      else if (d.rank2) parent = d.rank2.value.replace(nodePrefix, "");
      else if (d.rank3) parent = d.rank3.value.replace(nodePrefix, "");
      else if (d.rank4) parent = d.rank4.value.replace(nodePrefix, "");
      else if (d.rank5) parent = d.rank5.value.replace(nodePrefix, "");
      else if (d.rank6) parent = d.rank6.value.replace(nodePrefix, "");
      else if (d.rank7) parent = d.rank7.value.replace(nodePrefix, "");
      tree.push({
        id: d.taxon.value.replace(idPrefix, ""),
        label: d.taxon_label.value,
        parent: parent
      }) 
    }
    for (let i = 1; i <= 7; i++) {
      let rank = "rank" + i;
      if (d[rank] && !checked[d[rank].value]) {
        checked[d[rank].value] = true;
        let label = d["rank" + i + "_label"].value;
        let parent = "root";
        for (let j = i + 1; j <= 7; j++) {
          if (d["rank" + j]) {
            parent = d["rank" + j].value.replace(nodePrefix, "");
            break;
          }
        }
        tree.push({
          id: d[rank].value.replace(nodePrefix, ""),
          label: label,
          parent: parent
        })
      }
    }
  })
  leaf.results.bindings.forEach(d => {
    tree.push({
      id: d.sample.value.replace(idPrefix, ""),
      label: d.sample_label.value,
      parent: d.taxon.value.replace(nodePrefix, ""),
      leaf: true
    }) 
  })

  return tree;
}
```