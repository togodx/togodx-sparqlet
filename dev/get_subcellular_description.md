# アミノ酸残基の位置から、細胞内局在に関する情報をUniProtから取得する （山本）

## Description

## Endpoint

https://integbio.jp/rdf/sib/sparql

## Parameters

* `uniprot_id` ( UniProt ID )
  * default: Q8TCT9
  * examples: P10275, ...
* `position` ( Residue Position )
  * default: 10
  * examples: 25, 77

## `protein_id`
```javascript
({uniprot_id})=>{
  return "<http://purl.uniprot.org/uniprot/" + uniprot_id + ">"
}
```

## `Query`

```sparql
PREFIX up: <http://purl.uniprot.org/core/>
PREFIX skos: <http://www.w3.org/2004/02/skos/core#>
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX faldo: <http://biohackathon.org/resource/faldo#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>

SELECT ?protein ?position ?atype_label ?description ?begin ?end
WHERE {
  VALUES ?protein { {{protein_id}} }
  VALUES ?position { {{position}} }
  VALUES ?annot_type { up:Transmembrane_Annotation up:Topological_Domain_Annotation }
  ?protein up:annotation [
      up:range ?range ;
      rdfs:comment ?description ;
      a ?annot_type ] .
  ?annot_type rdfs:label ?atype_label .
  ?range (faldo:begin / faldo:position) ?begin ;
         (faldo:end / faldo:position) ?end .
  FILTER (?begin <= ?position && ?position <= ?end)
}
```