# uniprot mass “ñ€ŠÖŒWiç‰®j

## Endpoint
https://integbio.jp/togosite/sparql

## `data`
```sparql
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX taxon: <http://purl.uniprot.org/taxonomy/>
PREFIX up: <http://purl.uniprot.org/core/>
PREFIX uniprot: <http://purl.uniprot.org/uniprot/>
PREFIX xsd: <http://www.w3.org/2001/XMLSchema#>
SELECT DISTINCT ?uniprot ?label ?mass        
FROM <http://rdf.integbio.jp/dataset/togosite/uniprot>
WHERE {
  ?uniprot a up:Protein ;
           up:mnemonic ?label ;
           up:organism taxon:9606 ;
           up:proteome ?proteome ;
           up:sequence/up:mass ?mass .
  FILTER(REGEX(STR(?proteome), "UP000005640"))
}
```

## `return`
```javascript
({data})=>{
  const idPrefix = "http://purl.uniprot.org/uniprot/";
  
  let tree = [];
  data.results.bindings.map(d => {
    const num = parseInt(Number(d.mass.value) / 10000) * 10;
    const bin_id = num + "-" + (num + 10);
    tree.push({
      id: d.uniprot.value.replace(idPrefix, ""),
      label: d.label.value,
      value: Number(d.mass.value),
      binId: bin_id,
      binLabel: bin_id + " kDa"
    })
  })
  
  return tree;
}
```