# UniProt helical regions（井手・千葉）* alpha helixの合計長をタンパク質全長における割合で示す

## Description
- Data sources
    - [UniProt](https://www.uniprot.org/)

- Query
    - Input
        - UniProt ID
    - Output
        - Regions of Helix_Annotation in UniProt

## Parameters
* `categoryIds` -(type:Annotation)
  * default: Helix_Annotation
  * example: Beta_Strand_Annotation

## `annotation_types`
- annotation 選択
```javascript
({ categoryIds }) => {
  categoryIds = categoryIds.replace(/,/g," ")
  if (categoryIds.match(/[^\s]/)) {
    return categoryIds.split(/\s+/);
  }
}
```

## Endpoint
https://integbio.jp/togosite/sparql

## `main`
```sparql
PREFIX up: <http://purl.uniprot.org/core/>
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX faldo: <http://biohackathon.org/resource/faldo#>

SELECT DISTINCT ?uniprot ?mnemonic ?seq_length ?region_length ?begin_position ?end_position
FROM <http://rdf.integbio.jp/dataset/togosite/uniprot>
WHERE {
  VALUES ?annotation_type { {{#each annotation_types}} up:{{this}} {{/each}} } 
  ?uniprot up:proteome ?proteome_subset ;
      up:mnemonic ?mnemonic;
      up:annotation ?annotation .
  ?annotation a ?annotation_type ;
      up:range ?range .
  ?range faldo:begin/faldo:position ?begin_position ;
         faldo:end/faldo:position ?end_position .
  ?range faldo:begin/faldo:reference/rdf:value ?seq .
  BIND(STRLEN(?seq) AS ?seq_length)
  BIND((?end_position - ?begin_position + 1) AS ?region_length)
  FILTER(REGEX(STR(?proteome_subset), "UP000005640"))
}
ORDER BY ?uniprot ?begin_position
```

## `results`

```javascript
({ main }) => {
  let tree = [];
  let id_match;
  main.results.bindings.map(d => {
    let total_length;
    if (id_match === d.uniprot.value){
      total_length = total_length + Number(d.region_length.value);
      tree.pop();
    } else{  
      id_match = d.uniprot.value;
      total_length = Number(d.region_length.value);
    }
    const num = parseInt( 100* total_length/Number(d.seq_length.value));
    tree.push({
      id: d.uniprot.value.replace('http://purl.uniprot.org/uniprot/', ''),
      label: d.mnemonic.value,
      value: num ,
      binId: num + 1 ,
      binLabel: num + "%"
    })
  });
  return tree;
}
```