# 日本の難病疾患のカテゴリフィルター（NANDO階層利用）(高月)作業中
 
## Description

- Data sources
    - Nanbyo Disease Ontology (NANDO):[http://nanbyodata.jp/ontology/nando](http://nanbyodata.jp/ontology/nando)
- Input/Output
     -  Input
        - NANDO ID
    - Output
        - NANDO category
- Supplementary information


## Endpoint

https://integbio.jp/togosite/sparql

## `data`

```sparql
PREFIX nando: <http://nanbyodata.jp/ontology/NANDO_>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>

SELECT DISTINCT ?id ?label ?parent
WHERE {
 GRAPH <http://rdf.integbio.jp/dataset/togosite/nando> { 
    ?id rdfs:label ?label.
    FILTER (lang(?label) = "en")
    ?id rdfs:subClassOf ?parent.
  }}

```
## `return`

```javascript
({ data, mode }) => {
  if (mode === "idList") {
    return Array.from(new Set(
      data.results.bindings.map((d) =>
        d.nando.value.replace("http://nanbyodata.jp/ontology/NANDO_", "")
      )
    ));
  } else if (mode === "objectList") {
    return data.results.bindings.map((d) => ({
      id: d.nando.value.replace("http://nanbyodata.jp/ontology/NANDO_", ""),
      attribute: {
        categoryId: d.category.value.replace("http://nanbyodata.jp/ontology/NANDO_", ""),
        uri: d.category.value,
      label: d.nando_label.value
      }
    }));
  } else {
    return data.results.bindings.map((d) => ({
      categoryId: d.category.value
        .replace("http://nanbyodata.jp/ontology/NANDO_", ""),
      label: d.nando_label.value,
      count: Number(d.count.value),
      hasChild: Boolean(d.child)
    }));
  }
};
```


## MEMO
-Author
 - Takatsuki