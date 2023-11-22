# Isoform specific GO annotation existence from nextprot (守屋)

## Endpoint
{{SPARQLIST_TOGODX_SPARQL}}

## `hasSpecific`
* SPECIFIC フラグがなくなったので、数を数えて　isoform 間の違いを検出
```sparql
PREFIX : <http://nextprot.org/rdf#>
PREFIX skos: <http://www.w3.org/2004/02/skos/core#>
PREFIX core: <http://purl.uniprot.org/core/>
SELECT DISTINCT ?np ?label ?go
FROM <http://rdf.integbio.jp/dataset/togosite/nextprot>
FROM <http://rdf.integbio.jp/dataset/togosite/uniprot>
WHERE {
      {
        SELECT ?np ?label ?go ?go_term (COUNT (DISTINCT ?iso) AS ?go_count) ?iso_count
        {
          VALUES ?go { :GoBiologicalProcess :GoCellularComponent :GoMolecularFunction }
          ?np a :Entry ;
              skos:exactMatch/core:mnemonic ?label ;
              :isoform ?iso .
          ?iso :generalAnnotation [
            a ?go ; 
            :term ?go_term
          ] . 
          {
            SELECT ?np (COUNT (?iso) AS ?iso_count)
            {
              ?np a :Entry ;
                  :isoform ?iso.
            }
          }
        }
    }
   FILTER (?iso_count > ?go_count)  # compare #isoforms : #isoforms_with_annotation
}
```

## `hasIsoform`
```sparql
PREFIX : <http://nextprot.org/rdf#>
PREFIX skos: <http://www.w3.org/2004/02/skos/core#>
PREFIX core: <http://purl.uniprot.org/core/>
SELECT ?np ?label
WHERE {
  {
    SELECT DISTINCT ?np ?label (COUNT (?iso) AS ?iso_count) 
    FROM <http://rdf.integbio.jp/dataset/togosite/nextprot>
    FROM <http://rdf.integbio.jp/dataset/togosite/uniprot>
    WHERE {
      ?np a :Entry ;
          skos:exactMatch/core:mnemonic ?label ;
          :isoform ?iso .
    }
  }
  FILTER (?iso_count > 1)
}
```


## `return`
```javascript
({hasSpecific, hasIsoform})=>{
  let idPrefix = "http://nextprot.org/rdf/entry/NX_";
  let nodePrefix = "http://nextprot.org/rdf#";
  let withId = "hasSpecific";
  let withoutId = "unclassified";
  
  let tree = [
    {
      id: "root",
      label: "root node",
      root: true
    },{
      id: "GoBiologicalProcess",
      label: "With biological process specifcity",
      parent: "root"
    },{
      id: "GoCellularComponent",
      label: "With cellular component specifcity",
      parent: "root"
    },{
      id: "GoMolecularFunction",
      label: "With molecular function specifcity",
      parent: "root"
    },{
      id: withoutId,
      label: "Unknown",
      parent: "root"
    }
  ];

  let specific = {};
  hasSpecific.results.bindings.map(d => {
    specific[d.np.value] = true;
    tree.push({
      id: d.np.value.replace(idPrefix, ""),
      label: d.label.value,
      parent: d.go.value.replace(nodePrefix, ""),
      leaf: true
    });
  });
  hasIsoform.results.bindings.map(d => {
    if (!specific[d.np.value]) {
      tree.push({
        id: d.np.value.replace(idPrefix, ""),
        label: d.label.value,
        parent: withoutId,
        leaf: true
      });
    }
  });

  return tree;
}
```