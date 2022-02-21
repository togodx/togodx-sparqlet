#  Taxonomy of human-hosted microbe from MicrobeDB.jp

## Endpoint

https://microbedb.jp/sparql

## `prokaryotes`
```sparql
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX sio: <http://semanticscience.org/resource/>
PREFIX mdbv: <http://purl.jp/bio/11/mdbv#>
PREFIX obo: <http://purl.obolibrary.org/obo/>
PREFIX tax: <http://ddbj.nig.ac.jp/ontologies/taxonomy/>
PREFIX idtax: <http://identifiers.org/taxonomy/>
SELECT DISTINCT ?species ?species_label ?genus ?genus_label ?order ?order_label ?phylum ?phylum_label ?domain ?domain_label
WHERE {
  [] a sio:SIO_001050 ;
     sio:SIO_000008 [
       a mdbv:HostName;
       sio:SIO_000300 "Homo sapiens"
     ] ;
     obo:RO_0002162 ?species .
  ?species rdfs:label ?species_label_pre ;
           rdfs:subClassOf* ?genus .
  ?genus tax:rank tax:Genus ;
         rdfs:label ?genus_label ;
         rdfs:subClassOf* ?order .
  ?order tax:rank tax:Order ;
         rdfs:label ?order_label ;
         rdfs:subClassOf* ?phylum .
  ?phylum tax:rank tax:Phylum ;
          rdfs:label ?phylum_label ;
          rdfs:subClassOf* ?domain .
  ?domain tax:rank tax:Superkingdom ;
          rdfs:label ?domain_label .
  FILTER ( ?domain != idtax:2759 ) # prokaryotes (not eukaryotes)
  MINUS { ?species rdfs:subClassOf* idtax:408169 } # w/o metagenome
  BIND(STR(?species_label_pre) AS ?species_label) # uniform with/without xsd:string datatype
}
```

## `eukaryotes`
```sparql
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX sio: <http://semanticscience.org/resource/>
PREFIX mdbv: <http://purl.jp/bio/11/mdbv#>
PREFIX obo: <http://purl.obolibrary.org/obo/>
PREFIX tax: <http://ddbj.nig.ac.jp/ontologies/taxonomy/>
PREFIX idtax: <http://identifiers.org/taxonomy/>
SELECT DISTINCT ?species ?species_label ?genus ?genus_label ?order ?order_label ?phylum ?phylum_label ?kingdom ?kingdom_label ?domain ?domain_label
WHERE {
  [] a sio:SIO_001050 ;
       sio:SIO_000008 [
         a mdbv:HostName;
         sio:SIO_000300 "Homo sapiens"
       ] ;
          obo:RO_0002162 ?species .
  ?species rdfs:label ?species_label_pre ;
           rdfs:subClassOf* ?genus .
  ?genus tax:rank tax:Genus ;
         rdfs:label ?genus_label ;
         rdfs:subClassOf* ?order .
  ?order tax:rank tax:Order ;
         rdfs:label ?order_label ;
         rdfs:subClassOf* ?phylum .
  ?phylum tax:rank tax:Phylum ;
          rdfs:label ?phylum_label ;
          rdfs:subClassOf* ?kingdom .
  ?kingdom tax:rank tax:Kingdom ;
          rdfs:label ?kingdom_label ;
           rdfs:subClassOf* ?domain .
  ?domain tax:rank tax:Superkingdom ;
          rdfs:label ?domain_label .
  FILTER ( ?domain = idtax:2759 ) # eukaryotes
  MINUS { ?species rdfs:subClassOf* idtax:408169 } # w/o metagenome
  BIND(STR(?species_label_pre) AS ?species_label) # uniform with/without xsd:string datatype
}
```

## `return`
```javascript
({prokaryotes, eukaryotes}) => {
  const idPrefix = "http://identifiers.org/taxonomy/";

  let tree = [
    {
      id: "root",
      label: "organism",
      root: true
    }
  ];

  let checked = {};
  // prokaryotes
  prokaryotes.results.bindings.forEach(d => {
    if (!checked[d.species.value]) {
      checked[d.species.value] = true;
      tree.push({
        id: d.species.value.replace(idPrefix, ""),
        label: d.species_label.value,
        parent: d.genus.value.replace(idPrefix, ""),
        leaf: true
      }) 
    }
    if (!checked[d.genus.value]) {
      checked[d.genus.value] = true;
      tree.push({
        id: d.genus.value.replace(idPrefix, ""),
        label: d.genus_label.value,
        parent: d.order.value.replace(idPrefix, "")
      })
    }
    if (!checked[d.order.value]) {
      checked[d.order.value] = true;
      tree.push({
        id: d.order.value.replace(idPrefix, ""),
        label: d.order_label.value,
        parent: d.phylum.value.replace(idPrefix, "")
      })
    }
    if (!checked[d.phylum.value]) {
      checked[d.phylum.value] = true;
      tree.push({
        id: d.phylum.value.replace(idPrefix, ""),
        label: d.phylum_label.value,
        parent: d.domain.value.replace(idPrefix, "")
      })
    }
    if (!checked[d.domain.value]) {
      checked[d.domain.value] = true;
      tree.push({
        id: d.domain.value.replace(idPrefix, ""),
        label: d.domain_label.value,
        parent: "root"
      })
    } 
  })
  
  // eukaryotes
  eukaryotes.results.bindings.forEach(d => {
    if (!checked[d.species.value]) {
      checked[d.species.value] = true;
      tree.push({
        id: d.species.value.replace(idPrefix, ""),
        label: d.species_label.value,
        parent: d.genus.value.replace(idPrefix, ""),
        leaf: true
      })
    }
    if (!checked[d.genus.value]) {
      checked[d.genus.value] = true;
      tree.push({
        id: d.genus.value.replace(idPrefix, ""),
        label: d.genus_label.value,
        parent: d.order.value.replace(idPrefix, "")
      })
    }
    if (!checked[d.order.value]) {
      checked[d.order.value] = true;
      tree.push({
        id: d.order.value.replace(idPrefix, ""),
        label: d.order_label.value,
        parent: d.phylum.value.replace(idPrefix, "")
      })
    }
    if (!checked[d.phylum.value]) {
      checked[d.phylum.value] = true;
      tree.push({
        id: d.phylum.value.replace(idPrefix, ""),
        label: d.phylum_label.value,
        parent: d.kingdom.value.replace(idPrefix, "")
      })
    }
    if (!checked[d.kingdom.value]) {
      checked[d.kingdom.value] = true;
      tree.push({
        id: d.kingdom.value.replace(idPrefix, ""),
        label: d.kingdom_label.value,
        parent: d.domain.value.replace(idPrefix, "")
      })
    }
    if (!checked[d.domain.value]) {
      checked[d.domain.value] = true;
      tree.push({
        id: d.domain.value.replace(idPrefix, ""),
        label: d.domain_label.value,
        parent: "root"
      })
    }
  })

  return tree;
}
```