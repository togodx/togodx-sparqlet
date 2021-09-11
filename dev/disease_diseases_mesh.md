# Diseases in MeSH （三橋）

## Description

- Data sources
    -  [Medical Subject Headings (MeSH)](https://www.nlm.nih.gov/mesh/meshhome.html) 
- Query
    - Input
        - MeSH Descriptor
    - Output
        -  [Diseases ([C])](https://meshb.nlm.nih.gov/treeView) and its subcategories of MeSH

## Parameters

* `root` (type: MeSH) (Req.)
  * default: C
  * example: C (Diseases) 、C04  (Neoplasms) https://meshb.nlm.nih.gov/record/ui?ui=D009369

## Endpoint
https://integbio.jp/togosite/sparql

## `data`
```sparql
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX mesh: <http://id.nlm.nih.gov/mesh/>
PREFIX meshv: <http://id.nlm.nih.gov/mesh/vocab#>
PREFIX tree: <http://id.nlm.nih.gov/mesh/>

SELECT ?mesh ?tree ?label
FROM <http://rdf.integbio.jp/dataset/togosite/mesh>
WHERE {
  # MeSH TreeのRoot(Diseases[C]) のURI もラベルもないので、その下の階層(Infections[C01],...)のDescriptor(D007239)を列挙する
  # See https://meshb.nlm.nih.gov/treeView
  VALUES ?mesh { mesh:D007239 mesh:D009369 mesh:D009140 mesh:D004066 mesh:D009057 mesh:D012140 mesh:D010038 mesh:D009422 mesh:D005128 mesh:D052801 mesh:D005261 mesh:D002318 mesh:D006425 mesh:D009358 mesh:D017437 mesh:D009750 mesh:D004700 mesh:D0071154 mesh:D007280 mesh:D013568 mesh:D009784 }

  ?tree ^meshv:treeNumber/rdfs:label ?label .
  ?mesh meshv:treeNumber/meshv:parentTreeNumber* ?tree .
  FILTER(lang(?label) = "en")
}
```