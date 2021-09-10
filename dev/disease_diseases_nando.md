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
- categoryId があった場合に絞り込み
- queryIds があった場合に絞り込み
```sparql

PREFIX nando: <http://nanbyodata.jp/ontology/NANDO_>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>

{{#if mode}}
SELECT DISTINCT ?nando ?category STR(?label) AS ?nando_label
{{else}}
SELECT ?category STR(?label) AS ?nando_label (COUNT (DISTINCT ?nando) AS ?count) (SAMPLE(?child_category) AS ?child)
{{/if}}
WHERE {
{{#if queryArray}}
  VALUES ?nando { {{#each queryArray}} nando:{{this}} {{/each}} }
{{/if}}
{{#if categoryArray}}
  {{#if mode}}
  VALUES ?category { {{#each categoryArray}} nando:{{this}} {{/each}} }    
  {{else}}
  VALUES ?parent { {{#each categoryArray}} nando:{{this}} {{/each}} }
  {{/if}}
{{/if}}
 GRAPH <http://rdf.integbio.jp/dataset/togosite/nando> { 
 {{#unless  mode}}
    ?category rdfs:subClassOf ?parent.
 {{/unless}}
    ?category rdfs:label ?label.
    FILTER (lang(?label) = "en")
    ?nando rdfs:subClassOf* ?category.
  }
  OPTIONAL {
    ?child_category rdfs:subClassOf ?category .
  }
} 
{{#unless mode}}  
  ORDER BY DESC(?count)
{{/unless}}
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