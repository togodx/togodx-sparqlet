# PDB alpha_helix distributeion (井手, 守屋）作業中

## Description

- Data sources
    - The number of alpha-helices recorded in the PDB entry.
    - This item based on the data of January 20, 2021 of PDBj. 
        - The latest data can be obtained from the URL below. https://data.pdbj.org/pdbjplus/data/pdb/rdf/
- Query
    - Input
        - PDB ID
    - Output
        - The alpha-helix value contained in each entry.

## Endpoint

https://integbio.jp/togosite/sparql

## `withAnnotation`

```sparql
PREFIX pdbr: <https://rdf.wwpdb.org/pdb/>
PREFIX pdbo: <https://rdf.wwpdb.org/schema/pdbx-v50.owl#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX dc: <http://purl.org/dc/elements/1.1/>
PREFIX xsd: <http://www.w3.org/2001/XMLSchema#>

SELECT DISTINCT ?leaf (COUNT(?helix) AS ?target_num) 
WHERE {
      ?leaf  a pdbo:datablock ;
                 pdbo:has_struct_confCategory ?helix .
      ?helix pdbo:has_struct_conf ?helix_each .
      ?leaf  dc:title ?label .  
      {
        SELECT DISTINCT ?leaf {
          ?leaf pdbo:has_entityCategory
                  / pdbo:has_entity
                  / rdfs:seeAlso <http://identifiers.org/taxonomy/9606> .
        }
      }
}
```

## `withoutAnnotation`
- ヘリックスを持たないタンパク質の数
```sparql
PREFIX pdbr: <https://rdf.wwpdb.org/pdb/>
PREFIX pdbo: <https://rdf.wwpdb.org/schema/pdbx-v50.owl#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX dc: <http://purl.org/dc/elements/1.1/>
PREFIX xsd: <http://www.w3.org/2001/XMLSchema#> 
    SELECT DISTINCT ?PDBentry ?title 
    WHERE {
      {{#if queryArray}}
      VALUES ?PDBentry { {{#each queryArray}} pdbr:{{this}} {{/each}} }
      {{/if}}
      ?PDBentry  a pdbo:datablock ;
                 dc:title ?title .
      MINUS {?PDBentry pdbo:has_struct_confCategory ?helix . }
      {
        SELECT DISTINCT ?PDBentry {
          ?PDBentry pdbo:has_entityCategory
                  / pdbo:has_entity
                  / rdfs:seeAlso <http://identifiers.org/taxonomy/9606> .
        }
      }
    }
  }
  BIND ("0" AS ?target_num)
{{/if}}
}
```

## `return`
```javascript
({mode, queryIds, categoryIds, withTarget, withoutTarget})=>{
  // renge
  let range = {begin: 0, end: Infinity};
  let cids = false;
  if (categoryIds) {
    if (categoryIds.match(/^[\d\.]+-/)) range.begin = Number(categoryIds.match(/^([\d\.]+)-/)[1]);
    if (categoryIds.match(/-[\d\.]+$/)) range.end = Number(categoryIds.match(/-([\d\.]+)$/)[1]);
    if (categoryIds.match(/^[^-]+$/)) {
      cids = categoryIds.split(/,/).reduce((obj, a)=>{
        obj[Number(a)] = true;
        return obj;
      }, {});
    }
  }
  
  if (mode) {
    const idVarName = "PDBentry";
    const idPrefix = "https://rdf.wwpdb.org/pdb/";
    // add without-binding-site-data
    if (withoutTarget.results.bindings[0] && withoutTarget.results.bindings[0][idVarName]) withTarget.results.bindings = withTarget.results.bindings.concat(withoutTarget.results.bindings);
    let filteredData = [];
    if (categoryIds) {
      for (let d of withTarget.results.bindings) {
        const num = Number(d.target_num.value);
        if ((!cids && range.begin <= num && num <= range.end)
            || (cids[num])) {
          filteredData.push(d);
        }
      }
    } else filteredData = withTarget.results.bindings;
    if (mode == "objectList") return filteredData.map(d=>{
      return {
        id: d[idVarName].value.replace(idPrefix, ""),
        attribute: {categoryId: d.target_num.value, label: d.target_num.value}
      }
    });
    if (mode == "idList") return filteredData.map(d=>d[idVarName].value.replace(idPrefix, ""));
  }

  if (!queryIds || withoutTarget.results.bindings[0].count.value != 0) {
    withTarget.results.bindings.unshift( {count: {value: withoutTarget.results.bindings[0].count.value}, target_num: {value: "0"}}  ); // カウント 0 を追加
  }
  let value = 0;
  let res = [];
  for (let d of withTarget.results.bindings) {
    const num = Number(d.target_num.value);
    if (value < num) {
      // fill missing value by 0
      for (let emptyValue = value; emptyValue < num; emptyValue++) {
        if ((!queryIds) 
            && ((!categoryIds) || ((!cids && range.begin <= emptyValue && emptyValue <= range.end) || (cids[emptyValue])))) {
        	res.push( { categoryId: emptyValue.toString(), label: emptyValue.toString(), count: 0} );
        }
      }
    }
    value = num + 1;
    if ((!categoryIds)
      || ((!cids && range.begin <= num && num <= range.end) || (cids[num]))) {
      res.push( { categoryId: d.target_num.value, label: d.target_num.value, count: Number(d.count.value)} );
    }
  }       
  return res;
}
```