# Ensembl transcript length （池田）

## Description

- Data sources
    - Ensembl human release 107: [http://nov2020.archive.ensembl.org/Homo_sapiens/Info/Index](http://nov2020.archive.ensembl.org/Homo_sapiens/Info/Index)

## Endpoint

https://integbio.jp/togosite/sparql

## `data1`
SPARQL 内での sum は遅いので、エキソン長の足し合わせは js で行う

取得件数が多く 1000000 制限に引っかかるので chromosome で二分割
```sparql
PREFIX obo: <http://purl.obolibrary.org/obo/>
PREFIX taxon: <http://identifiers.org/taxonomy/>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX faldo: <http://biohackathon.org/resource/faldo#>
PREFIX dc: <http://purl.org/dc/elements/1.1/>
PREFIX xsd: <http://www.w3.org/2001/XMLSchema#>

SELECT DISTINCT ?child ?child_label ?exon_len
FROM <http://rdf.integbio.jp/dataset/togosite/ensembl>
WHERE {
  ?enst obo:SO_transcribed_from ?ensg .
  ?ensg obo:RO_0002162 taxon:9606 ;
        faldo:location ?ensg_location .
  ?enst dc:identifier ?child ;
        rdfs:label ?child_label ;
        obo:SO_has_part ?ense .
  ?ense faldo:location ?ense_location .
  BIND(STRBEFORE(STRAFTER(STR(?ensg_location), "GRCh38/"), ":") AS ?chromosome)
  VALUES ?chromosome {
      "1" "2" "3" "4" "5" "6" "7" "8" "9" "10"
      #"11" "12" "13" "14" "15" "16" "17" "18" "19" "20" "21" "22"
      #"X" "Y" "MT"
  }
  BIND(STRBEFORE(STRAFTER(STRAFTER(STR(?ense_location), "GRCh38/"), ":"), "-") AS ?exon_start)
  BIND(STRBEFORE(STRAFTER(STRAFTER(STRAFTER(STR(?ense_location), "GRCh38/"), ":"), "-"), ":") AS ?exon_end)
  BIND((xsd:integer(?exon_end) - xsd:integer(?exon_start) + 1) AS ?exon_len)
}
```

## `data2`

```sparql
PREFIX obo: <http://purl.obolibrary.org/obo/>
PREFIX taxon: <http://identifiers.org/taxonomy/>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX faldo: <http://biohackathon.org/resource/faldo#>
PREFIX dc: <http://purl.org/dc/elements/1.1/>
PREFIX xsd: <http://www.w3.org/2001/XMLSchema#>

SELECT DISTINCT ?child ?child_label ?exon_len
FROM <http://rdf.integbio.jp/dataset/togosite/ensembl>
WHERE {
  ?enst obo:SO_transcribed_from ?ensg .
  ?ensg obo:RO_0002162 taxon:9606 ;
        faldo:location ?ensg_location .
  ?enst dc:identifier ?child ;
        rdfs:label ?child_label ;
        obo:SO_has_part ?ense .
  ?ense faldo:location ?ense_location .
  BIND(STRBEFORE(STRAFTER(STR(?ensg_location), "GRCh38/"), ":") AS ?chromosome)
  VALUES ?chromosome {
      #"1" "2" "3" "4" "5" "6" "7" "8" "9" "10"
      "11" "12" "13" "14" "15" "16" "17" "18" "19" "20" "21" "22"
      "X" "Y" "MT"
  }
  BIND(STRBEFORE(STRAFTER(STRAFTER(STR(?ense_location), "GRCh38/"), ":"), "-") AS ?exon_start)
  BIND(STRBEFORE(STRAFTER(STRAFTER(STRAFTER(STR(?ense_location), "GRCh38/"), ":"), "-"), ":") AS ?exon_end)
  BIND((xsd:integer(?exon_end) - xsd:integer(?exon_start) + 1) AS ?exon_len)
}
```

## `return`

```javascript
({data1, data2}) => {
  const data = data1.results.bindings.concat(data2.results.bindings)
  let len_sum = {};

  data.forEach(d => {
    if (!len_sum[d.child.value]) {
      len_sum[d.child.value] = {
        len: Number(d.exon_len.value),
        label: d.child_label.value
      };
    }
    else {
      len_sum[d.child.value].len += Number(d.exon_len.value);
    }
  });
  let transcripts = Object.keys(len_sum);
  const bin = 50;
  let tree = transcripts.map(t => {
    const num = bin * parseInt(len_sum[t].len/bin);
    const bin_id = num + "-" + (num + bin - 1);
    return {
      id: t,
      label: len_sum[t].label,
      value: len_sum[t].len,
      binId: bin_id,
      binLabel: bin_id
    };
  });
  return tree;
};
```
