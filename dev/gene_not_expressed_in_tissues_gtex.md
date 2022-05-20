# Tissues where a gene is not expressed （池田）

## Description

- Data sources
    - [GTEx version 8](https://gtexportal.org/home/datasets)

## Endpoint

https://integbio.jp/togosite/sparql

## `data`
```sparql
PREFIX obo: <http://purl.obolibrary.org/obo/>
PREFIX enso: <http://rdf.ebi.ac.uk/terms/ensembl/>
PREFIX dcterms: <http://purl.org/dc/terms/>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX refexo:  <http://purl.jp/bio/01/refexo#>
PREFIX sio: <http://semanticscience.org/resource/>
PREFIX schema: <http://schema.org/>
PREFIX ensg: <http://rdf.ebi.ac.uk/resource/ensembl/>

SELECT DISTINCT ?parent_label ?parent ?child ?child_label
WHERE {
  GRAPH <http://rdf.integbio.jp/dataset/togosite/refex_gtex_v8_summary> {
    ?refex a refexo:RefExEntry ;
           sio:SIO_000216 [
             a refexo:logTPMMax ;
             sio:SIO_000300 0
             #sio:SIO_000300 ?logtpmmax
           ] ;
           refexo:isMeasurementOf ?child ;
           refexo:refexSample ?refexs .
    #FILTER(?logtpmmax >= 0 && ?logtpmmax < 1)
  }
  
  GRAPH <http://rdf.integbio.jp/dataset/togosite/refexsample_gtex_v8_summary> {
    ?refexs dcterms:description ?parent_label ;
            dcterms:identifier ?parent .
  }

  GRAPH <http://rdf.integbio.jp/dataset/togosite/ensembl> {
    ?child a ?type ;
           rdfs:label ?child_label .
    # GTEx のプロトコルの原理上、small RNA は取れていないはずだが、GTEx のデータ上には存在してしまっている。
    # そのような RNA を GTEx のデータを用いて「無発現」と判断するべきでないので、除く。
    VALUES ?type {
      enso:protein_coding enso:lncRNA enso:pseudogene
      enso:polymorphic_pseudogene enso:processed_pseudogene enso:unitary_pseudogene enso:unprocessed_pseudogene
      enso:transcribed_processed_pseudogene enso:transcribed_unitary_pseudogene enso:transcribed_unprocessed_pseudogene
      enso:translated_processed_pseudogene enso:translated_unprocessed_pseudogene
      enso:IG_C_gene enso:IG_D_gene enso:IG_J_gene enso:IG_V_gene
      enso:IG_pseudogene enso:IG_C_pseudogene enso:IG_J_pseudogene enso:IG_V_pseudogene
      enso:TR_C_gene enso:TR_D_gene enso:TR_J_gene enso:TR_V_gene
      enso:TR_J_pseudogene enso:TR_V_pseudogene
      enso:TEC
    }
  }
}
```

## `all`
```sparql
PREFIX obo: <http://purl.obolibrary.org/obo/>
PREFIX so: <http://purl.obolibrary.org/obo/so#>
PREFIX enso: <http://rdf.ebi.ac.uk/terms/ensembl/>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX refexo:  <http://purl.jp/bio/01/refexo#>

SELECT DISTINCT ?child ?child_label
WHERE {
  GRAPH <http://rdf.integbio.jp/dataset/togosite/refex_gtex_v8_summary> {
    ?refex a refexo:RefExEntry ;
           refexo:isMeasurementOf ?child .
  }
  GRAPH <http://rdf.integbio.jp/dataset/togosite/ensembl> {
    ?child a ?type ;
           rdfs:label ?child_label .
    [] so:transcribed_from ?child .
    VALUES ?type {
      enso:protein_coding enso:lncRNA enso:pseudogene
      enso:polymorphic_pseudogene enso:processed_pseudogene enso:unitary_pseudogene enso:unprocessed_pseudogene
      enso:transcribed_processed_pseudogene enso:transcribed_unitary_pseudogene enso:transcribed_unprocessed_pseudogene
      enso:translated_processed_pseudogene enso:translated_unprocessed_pseudogene
      enso:IG_C_gene enso:IG_D_gene enso:IG_J_gene enso:IG_V_gene
      enso:IG_pseudogene enso:IG_C_pseudogene enso:IG_J_pseudogene enso:IG_V_pseudogene
      enso:TR_C_gene enso:TR_D_gene enso:TR_J_gene enso:TR_V_gene
      enso:TR_J_pseudogene enso:TR_V_pseudogene
      enso:TEC
    }
  }
}
```

## `return`

```javascript
({data, all}) => {
  const idPrefix = "http://rdf.ebi.ac.uk/resource/ensembl/";

  let tree = [{
    id: "root",
    label: "root node",
    root: true
  }];
  let chk = {};
  let unclassified = {};
  all.results.bindings.forEach(d => {
    let label = d.child_label.value;
    if (label == "") {
      label = d.child.value.replace(idPrefix, "");
    }
    unclassified[d.child.value] = label;
  });
  data.results.bindings.forEach(d => {
    if (!chk[d.parent.value]) {
      chk[d.parent.value] = true;
      tree.push({     
        id: d.parent_label.value,
        label: d.parent_label.value,
        leaf: false,
        parent: "root"
      });
    }
    let label = d.child_label.value;
    if (label == "") {
      label = d.child.value.replace(idPrefix, "");
    }
    tree.push({
      id: d.child.value.replace(idPrefix, ""),
      label: label,
      leaf: true,
      parent: d.parent_label.value
    });
    delete unclassified[d.child.value];
  });

  tree.push({     
    id: "unclassified",
    label: "Expressed in all tissues",
    leaf: false,
    parent: "root"
  });

  Object.keys(unclassified).forEach(d => {
    tree.push({
      id: d.replace(idPrefix, ""),
      label: unclassified[d],
      leaf: true,
      parent: "unclassified"
    });
  });
  return tree;
};

```
