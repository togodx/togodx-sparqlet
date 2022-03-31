# 理研BRCのマウス情報　作出方法別（高月）

## Description

- Data sources
     - RIKEN BRC Experimental Animal Division: [https://mus.brc.riken.jp/en/](https://mus.brc.riken.jp/en/)
- Input/Output
     -  Input
        - BRC No.
    - Output
        - Strain Type
- Supplementary information
     - ×××
     - RIKEN BRC実験動物開発室から提供されているマウスのリソース情報をタイプ別で分類


## Endpoint

https://knowledge.brc.riken.jp/sparql

```sparql

PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX brs: <http://metadb.riken.jp/db/bioresource_schema/brs_>

SELECT DISTINCT ?brcID ?category_label 
WHERE {
   GRAPH <http://metadb.riken.jp/db/xsearch_animal> { 
    ?brcID brs:strainTypeLink ?category.
    ?category rdfs:label ?category_label.
  }
} 
 ORDER BY DESC(?count)
```