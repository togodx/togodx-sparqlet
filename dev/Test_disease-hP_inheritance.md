# Human phenotype ontorogy inheritance 分類

## Description

- Data sources
    -  [Human Phenotype Ontology (HPO)](https://hpo.jax.org/app/) 
- Query
    - Input
        - HPO id
    - Output
        -  [Mode of inheritance (HP:0000005)](http://purl.obolibrary.org/obo/HP_0000005)  and its subcategories of HPO

## Endpoint

https://integbio.jp/rdf/sparql

## `graph`
```sparql

PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX dct: <http://purl.org/dc/terms/>
PREFIX obo: <http://purl.obolibrary.org/obo/>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX owl: <http://www.w3.org/2002/07/owl#>

SELECT DISTINCT ?hpid ?label ?parent
FROM <http://integbio.jp/rdf/ontology/hp>
WHERE {
  #root nodeはHP_000005
  VALUES ?root {  obo:HP_0000005  }    
  ?hpid rdfs:subClassOf+ ?root.
  ?hpid rdfs:label ?label.
  ?hpid rdfs:subClassOf ?parent.
  ?hpid rdf:type owl:Class.  
  # HP以外のIDも登録されているため、HPに限定するためにフィルターを追加
  FILTER regex(str(?hpid), "http://purl.obolibrary.org/obo/HP_" )
 } 
 GROUP BY ?hpid ?parent ?label

```
## `leaf`
```sparql
## endpoint https://integbio.jp/rdf/sparql

PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX obo: <http://purl.obolibrary.org/obo/>
PREFIX biolink: <https://w3id.org/biolink/vocab/>
PREFIX oboInOwl: <http://www.geneontology.org/formats/oboInOwl#>
 
SELECT DISTINCT ?mondo ?omim ?mondo_id ?omim_id ?omim_iri ?hp
 WHERE{ 
  GRAPH<http://integbio.jp/rdf/ontology/mondo>{
   ?mondo 
   oboInOwl:hasDbXref ?omim;
   oboInOwl:id ?mondo_id.
   FILTER (regex(?omim, 'OMIM'))
   BIND (REPLACE(str(?omim),"OMIM:","http://purl.obolibrary.org/obo/OMIM_") AS ?omim_id)
   BIND (IRI(?omim_id) AS ?omim_iri) }
  GRAPH <http://rdf.integbio.jp/dataset/monarch>{
   ?omim_iri
   biolink:has_mode_of_inheritance ?hp.
  }
}

```

## `return`

```javascript
({graph}) => {
  const idPrefix = "http://purl.obolibrary.org/obo/HP_";
  let tree = [
    {
      id: "0000005",
      label: "Mode of inheritance",
      root: true
    }
  ];
  graph.results.bindings.forEach(d => {
    tree.push({
      id: d.hpid.value.replace(idPrefix, ""),
      label: d.label.value,
      parent: d.parent.value.replace(idPrefix, "")
    });
  });
  return tree;
};
```

## MEMO
-Author
 - Takatsuki