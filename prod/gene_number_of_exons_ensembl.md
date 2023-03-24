# トランスクリプトをexon数ごとに分類（鈴木・八塚）

## Description

- Data sources
    - Ensembl human release 102: [http://nov2020.archive.ensembl.org/Homo_sapiens/Info/Index](http://nov2020.archive.ensembl.org/Homo_sapiens/Info/Index)
- Supplementary information
	- The number of exons for each transcript of each gene.
	- 各遺伝子の転写産物ごとにエクソン数をカウントした結果です。

## Endpoint

{{SPARQLIST_TOGODX_SPARQL}}

## `categoryArray`
```javascript
() => {
  let cArray = [];
  cArray.push(  {"binID":1, "min": 1, "max": 1},
                  {"binID":2, "min": 2, "max": 2},
                  {"binID":3, "min": 3, "max": 3},
                  {"binID":4, "min": 4, "max": 4},
                  {"binID":5, "min": 5, "max": 5},
                  {"binID":6, "min": 6, "max": 6},
                  {"binID":7, "min": 7, "max": 7},
                  {"binID":8, "min": 8, "max": 8},
                  {"binID":9, "min": 9, "max": 9},
                  {"binID":10, "min": 10, "max": 10},
                  {"binID":11, "min": 11, "max": 15},
                  {"binID":12, "min": 16, "max": 20},
                  {"binID":13, "min": 21, "max": 50},
                  {"binID":14, "min": 51, "max": 100},
                  {"binID":15, "min": 101, "max": 200},
                  {"binID":16, "min": 201, "max": 400},
                  {"binID":17, "min": 401, "max": 1000},
                  {"binID":18, "min": 1001,"max": 10000});
  return cArray;
}
```

## `query`
```sparql
# @endpoint https://integbio.jp/rdf/ebi/sparql
PREFIX obo:<http://purl.obolibrary.org/obo/>
PREFIX taxon:<http://identifiers.org/taxonomy/>
PREFIX enst:<http://rdf.ebi.ac.uk/resource/ensembl.transcript/>
PREFIX terms:<http://rdf.ebi.ac.uk/terms/ensembl/>
PREFIX faldo: <http://biohackathon.org/resource/faldo#>

SELECT DISTINCT ?enst_id ?exon_count ?bin_id
WHERE {
  VALUES (?bin_id ?min ?max) { {{#each categoryArray}}("{{this.binID}}-{{this.min}}-{{this.max}}" {{this.min}} {{this.max}}) {{/each}} }
  FILTER(?min <= ?exon_count &&  ?exon_count <= ?max){
     SELECT ?enst (COUNT(DISTINCT ?exon) AS ?exon_count)
      WHERE {
        ?enst obo:SO_has_part ?exon;
              obo:SO_transcribed_from ?ensg .
        ?ensg obo:RO_0002162 taxon:9606 ; # in taxon
              faldo:location ?ensg_location .
        BIND (strbefore(strafter(str(?ensg_location), "GRCh38/"), ":") AS ?chromosome)
        VALUES ?chromosome {
          "1" "2" "3" "4" "5" "6" "7" "8" "9" "10"
          "11" "12" "13" "14" "15" "16" "17" "18" "19" "20" "21" "22"
          "X" "Y" "MT"
        }
    }GROUP BY ?enst
  }
  BIND(REPLACE(STR(?enst), "http://rdf.ebi.ac.uk/resource/ensembl.transcript/", "") AS ?enst_id)
}

```

## `results`

```javascript
({query})=>{
  return query.results.bindings.map(d=>{
    var range = d.bin_id.value.split("-");  //
    return {
      id: d.enst_id.value, 
      label: d.enst_id.value,
      value: d.exon_count.value,
      binBegin: range[1],
      binEnd: range[2],
      binLabel: makeLabel(range[1], range[2])
    }
  });

  function makeLabel(num1, num2) {
    if (num1 == num2) {
      return num1;
     } else {
       return num1+"-"+num2 ;
     }
  }
};	
```
