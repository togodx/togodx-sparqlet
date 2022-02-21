# PlantGarden taxonomy-genome-chr-marker2 (信定)
- 生物種ごとのmarkerの分類 
- 生物種ーゲノムーchromosomeーmarker　で treeになっている

## Endpoint
https://mb2.ddbj.nig.ac.jp/sparql

## `main`

```sparql
prefix pg_ns: <https://plantgardden.jp/ns/>
prefix dcterms: <http://purl.org/dc/terms/>
prefix rdfs: <http://www.w3.org/2000/01/rdf-schema#>
select distinct ?marker_id ?marker_label ?chr_id  
from <http://plantgarden.jp/resource/pg_marker>
from <http://plantgarden.jp/resource/genome>
where {
?s a  pg_ns:Marker ;
dcterms:identifier ?marker_id ;
rdfs:label ?marker_label ;
pg_ns:genome ?genome ;
pg_ns:chr ?chr  .
?genome a  pg_ns:Genome ;
dcterms:identifier ?genome_id .
BIND ( CONCAT(?genome_id, ".", ?chr) as ?chr_id)
} 

```
## `return`
```javascript
({main}) => {
  
 let tree = [
    {
      id: "root",
      label: "root node",
      root: true
    }
  ];
  
  let edge = {};
 main.results.bindings.map(d => {
    tree.push({
      id: d.marker_id.value,
      label: d.marker_label.value,
      leaf: true,
      parent: d.chr_id.value
    })
      });
  
    return tree;
}
```