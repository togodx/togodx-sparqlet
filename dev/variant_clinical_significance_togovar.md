# variant clinical significance (三橋）

# variant

## Parameters

* `categoryIds` (type:Clinical significance from ClinVar)
  * default:uncertain_significance,likely_benign,benign,pathogenic,likely_pathogenic,conflicting_interpretations_of_pathogenicity,not_provided,benign_or_likely_benign,pathogenic_or_likely_pathogenic,other,drug_response,risk_factor
  * example:uncertain_significance,likely_benign,benign,pathogenic,likely_pathogenic,conflicting_interpretations_of_pathogenicity,not_provided,benign_or_likely_benign,pathogenic_or_likely_pathogenic,other,drug_response,risk_factor,association,affects,protective


## `categories`

```javascript
({categoryIds}) => {
  var id2label = {};
  id2label["affects"] = "Affects"
  id2label["association_not_found"] = "association not found"
  id2label["association"] = "association"
  id2label["benign_or_likely_benign_other"] = "Benign/Likely benign, other"
  id2label["benign_or_likely_benign_risk_factor"] = "Benign/Likely benign, risk factor"
  id2label["benign_or_likely_benign"] = "Benign/Likely benign"
  id2label["benign_other"] = "Benign, other"
  id2label["benign_risk_factor"] = "Benign, risk factor"
  id2label["benign"] = "Benign"
  id2label["confers_sensitivity"] = "confers sensitivity"
  id2label["conflicting_interpretations_of_pathogenicity_association_risk_factor"] = "Conflicting interpretations of pathogenicity, association, risk factor"
  id2label["conflicting_interpretations_of_pathogenicity_association"] = "Conflicting interpretations of pathogenicity, association"
  id2label["conflicting_interpretations_of_pathogenicity_other"] = "Conflicting interpretations of pathogenicity, other"
  id2label["conflicting_interpretations_of_pathogenicity_risk_factor"] = "Conflicting interpretations of pathogenicity, risk factor"
  id2label["conflicting_interpretations_of_pathogenicity"] = "Conflicting interpretations of pathogenicity"
  id2label["drug_response"] = "drug response"
  id2label["likely_benign_other"] = "Likely benign, other"
  id2label["likely_benign_risk_factor"] = "Likely benign, risk factor"
  id2label["likely_benign"] = "Likely benign"
  id2label["likely_pathogenic_affects"] = "Likely pathogenic, Affects"
  id2label["likely_pathogenic_other"] = "Likely pathogenic, other"
  id2label["likely_pathogenic_risk_factor"] = "Likely pathogenic, risk factor"
  id2label["likely_pathogenic"] = "Likely pathogenic"
  id2label["not_provided"] = "not provided"
  id2label["other_risk_factor"] = "other, risk factor"
  id2label["other"] = "other"
  id2label["pathogenic_affects"] = "Pathogenic, Affects"
  id2label["pathogenic_association"] = "Pathogenic, association"
  id2label["pathogenic_drug_response"] = "Pathogenic, drug response"
  id2label["pathogenic_or_likely_pathogenic_association"] = "Pathogenic/Likely pathogenic, association"
  id2label["pathogenic_or_likely_pathogenic_other"] = "Pathogenic/Likely pathogenic, other"
  id2label["pathogenic_or_likely_pathogenic_risk_factor"] = "Pathogenic/Likely pathogenic, risk factor"
  id2label["pathogenic_or_likely_pathogenic"] = "Pathogenic/Likely pathogenic"
  id2label["pathogenic_other"] = "Pathogenic, other"
  id2label["pathogenic_risk_factor"] = "Pathogenic, risk factor"
  id2label["pathogenic"] = "Pathogenic"
  id2label["protective"] = "protective"
  id2label["risk_factor"] = "risk factor"
  id2label["uncertain_significance_affects"] = "Uncertain significance, Affects"
  id2label["uncertain_significance_other"] = "Uncertain significance, other"
  id2label["uncertain_significance_risk_factor"] = "Uncertain significance, risk factor"
  id2label["uncertain_significance"] = "Uncertain significance"
  categoryIds = categoryIds.replace(/,/g," ")
  if (categoryIds.match(/[^\s]/)) return categoryIds.split(/\s+/).map( categoryId => id2label[categoryId]　);
  return false;
}
```

## Endpoint

https://integbio.jp/togosite/sparql

## `leaf`

```sparql
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX tgvo: <http://togovar.biosciencedbc.jp/vocabulary/>
PREFIX cvo: <http://purl.jp/bio/10/clinvar/>
PREFIX obo: <http://purl.obolibrary.org/obo/>
PREFIX dct: <http://purl.org/dc/terms/>
#SELECT count(DISTINCT ?tgv_id)
SELECT ?tgv_id ?rs_id ?category
FROM <http://rdf.integbio.jp/dataset/togosite/variation>
FROM <http://rdf.integbio.jp/dataset/togosite/variation/annotation/clinvar>
FROM <http://rdf.integbio.jp/dataset/togosite/clinvar>
WHERE {  
#  VALUES ?category { {{#each categories}} "{{this}}" {{/each}} }   
  VALUES ?catgory { "Uncertain significance" "Likely benign" "Benign" "Pathogenic" "Likely pathogenic" "Conflicting interpretations of pathogenicity" "not provided" "Benign/Likely benign" "Pathogenic/Likely pathogenic" "other" "drug response" "risk factor"}
  ?togovar dct:identifier ?tgv_id.
  ?togovar rdfs:seeAlso ?rs_id.
  ?togovar tgvo:condition/rdfs:seeAlso/cvo:interpreted_record/cvo:rcv_list/cvo:rcv_accession/cvo:interpretation ?category.  
}
limit 100000
```

## `return`

```javascript
({leaf}) => {
  const childLabelPrefix = "http://identifiers.org/dbsnp/";
  let tree = [
    {
      id: "root",
      label: "Clinvar root",
      root: true
    }
  ];
 var id2label = {};
  id2label["affects"] = "Affects"
  id2label["association_not_found"] = "association not found"
  id2label["association"] = "association"
  id2label["benign_or_likely_benign_other"] = "Benign/Likely benign, other"
  id2label["benign_or_likely_benign_risk_factor"] = "Benign/Likely benign, risk factor"
  id2label["benign_or_likely_benign"] = "Benign/Likely benign"
  id2label["benign_other"] = "Benign, other"
  id2label["benign_risk_factor"] = "Benign, risk factor"
  id2label["benign"] = "Benign"
  id2label["confers_sensitivity"] = "confers sensitivity"
  id2label["conflicting_interpretations_of_pathogenicity_association_risk_factor"] = "Conflicting interpretations of pathogenicity, association, risk factor"
  id2label["conflicting_interpretations_of_pathogenicity_association"] = "Conflicting interpretations of pathogenicity, association"
  id2label["conflicting_interpretations_of_pathogenicity_other"] = "Conflicting interpretations of pathogenicity, other"
  id2label["conflicting_interpretations_of_pathogenicity_risk_factor"] = "Conflicting interpretations of pathogenicity, risk factor"
  id2label["conflicting_interpretations_of_pathogenicity"] = "Conflicting interpretations of pathogenicity"
  id2label["drug_response"] = "drug response"
  id2label["likely_benign_other"] = "Likely benign, other"
  id2label["likely_benign_risk_factor"] = "Likely benign, risk factor"
  id2label["likely_benign"] = "Likely benign"
  id2label["likely_pathogenic_affects"] = "Likely pathogenic, Affects"
  id2label["likely_pathogenic_other"] = "Likely pathogenic, other"
  id2label["likely_pathogenic_risk_factor"] = "Likely pathogenic, risk factor"
  id2label["likely_pathogenic"] = "Likely pathogenic"
  id2label["not_provided"] = "not provided"
  id2label["other_risk_factor"] = "other, risk factor"
  id2label["other"] = "other"
  id2label["pathogenic_affects"] = "Pathogenic, Affects"
  id2label["pathogenic_association"] = "Pathogenic, association"
  id2label["pathogenic_drug_response"] = "Pathogenic, drug response"
  id2label["pathogenic_or_likely_pathogenic_association"] = "Pathogenic/Likely pathogenic, association"
  id2label["pathogenic_or_likely_pathogenic_other"] = "Pathogenic/Likely pathogenic, other"
  id2label["pathogenic_or_likely_pathogenic_risk_factor"] = "Pathogenic/Likely pathogenic, risk factor"
  id2label["pathogenic_or_likely_pathogenic"] = "Pathogenic/Likely pathogenic"
  id2label["pathogenic_other"] = "Pathogenic, other"
  id2label["pathogenic_risk_factor"] = "Pathogenic, risk factor"
  id2label["pathogenic"] = "Pathogenic"
  id2label["protective"] = "protective"
  id2label["risk_factor"] = "risk factor"
  id2label["uncertain_significance_affects"] = "Uncertain significance, Affects"
  id2label["uncertain_significance_other"] = "Uncertain significance, other"
  id2label["uncertain_significance_risk_factor"] = "Uncertain significance, risk factor"
  id2label["uncertain_significance"] = "Uncertain significance"
  // 親子関係
  for(let id in id2label){
    tree.push({
      id: id,
      label: id2label[id],
      parent: "root"
    })
  }
  // アノテーション関係
  leaf.results.bindings.map(d => {
    tree.push({
      id: d.tgv_id.value,
      label: d.rs_id.value.replace(childLabelPrefix, ""),
      leaf: true,
      parent: d.category.value.toLowerCase().replace("/", "_or_").replace(/,?\s+/g, "_")
    });
  })
  return tree;	
}
```