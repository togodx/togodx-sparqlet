# Variant report / Transcript

## Parameters

* `consequence_label` consequence_label
  * default: missense_variant

## Endpoint

https://grch38.togovar.org/sparql

## `result`

```sparql
PREFIX dct:  <http://purl.org/dc/terms/>
PREFIX dc11: <http://purl.org/dc/elements/1.1/>
PREFIX tgvo: <http://togovar.biosciencedbc.jp/vocabulary/>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>

SELECT ?transcript ?enst_id ?gene_symbol ?gene_xref ?hgvs_p ?hgvs_c ?sift ?polyphen ?consequence_label
WHERE {
    VALUES ?consequence_label {  "{{consequence_label}}" }

{
SELECT DISTINCT ?transcript ?enst_id ?gene_symbol ?gene_xref ?hgvs_p ?hgvs_c ?sift ?polyphen 
                (GROUP_CONCAT(DISTINCT ?_consequence_label ; separator = ",") AS ?consequence_label)
WHERE {  
  GRAPH <http://togovar.biosciencedbc.jp/variant> {
    ?variant dct:identifier ?tgv_id .
  }

  GRAPH <http://togovar.biosciencedbc.jp/variant/annotation/ensembl> {
    ?variant tgvo:hasConsequence ?_consequence .
    ?_consequence a ?_consequence_type .

    GRAPH <http://togovar.biosciencedbc.jp/so> {
      ?_consequence_type rdfs:label ?_consequence_label .
    }
    OPTIONAL { ?_consequence tgvo:sift ?sift . }
    OPTIONAL { ?_consequence tgvo:polyphen ?polyphen . }
    OPTIONAL {
      ?_consequence tgvo:hgvsp ?hgvs_p .
      FILTER STRSTARTS(?hgvs_p, 'ENSP')
    }
    OPTIONAL {
      ?_consequence tgvo:hgvsc ?hgvs_c .
      FILTER STRSTARTS(?hgvs_c, 'ENST')
    }
    OPTIONAL {
      ?_consequence tgvo:transcript ?transcript .
      OPTIONAL {
        GRAPH <http://togovar.biosciencedbc.jp/ensembl> {
          ?transcript dct:identifier|dc11:identifier ?enst_id .
        }
      }
    }
    OPTIONAL {
      ?_consequence tgvo:gene ?_gene .

      FILTER STRSTARTS(STR(?_gene), "http://rdf.ebi.ac.uk/resource/ensembl/ENSG")

      OPTIONAL {
        GRAPH <http://togovar.biosciencedbc.jp/ensembl> {
          ?_gene rdfs:label ?gene_symbol .
        }
      }
    }
    OPTIONAL {
      ?_consequence tgvo:hgnc ?gene_xref .
    }
  }
}
ORDER BY ?transcript
}
}
```