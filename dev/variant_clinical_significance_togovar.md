# variant clinical significance (三橋）

## Parameters

* `categoryIds` (type:Clinical significance from ClinVar)
  * example: uncertain_significance,likely_benign,benign,pathogenic,likely_pathogenic,conflicting_interpretations_of_pathogenicity,not_provided,benign_or_likely_benign,pathogenic_or_likely_pathogenic,other,drug_response,risk_factor,association,affects,protective

## `queryArray`
- Query TogoVarIDを配列に
```javascript
({queryIds}) => {
  queryIds = queryIds.replace(/,/g," ")
  if (queryIds.match(/[^\s]/)) return queryIds.split(/\s+/);
  return false;
}
```

## `categoryArray`
- Clinical siginificanceのIDをlabel(ClinVarの表記と同じ)に変換して配列に代入する。
  - [IDとClinVar表記の対応表](https://docs.google.com/spreadsheets/d/1qEy1uyS24AwlhfmNGdXWHLZv16ebvtCTa4W5dK28lwg/edit?usp=sharing)
  - [ClinVarのClinical significance一覧を取得するSPARQL](https://is.gd/01zgpr)
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

## `data`
```sparql

PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX tgvo: <http://togovar.biosciencedbc.jp/vocabulary/>
PREFIX cvo: <http://purl.jp/bio/10/clinvar/>
PREFIX obo: <http://purl.obolibrary.org/obo/>
PREFIX dct: <http://purl.org/dc/terms/>

SELECT ?id ?parent ?label SAMPLE(?child)
FROM <http://rdf.integbio.jp/dataset/togosite/variation>
FROM <http://rdf.integbio.jp/dataset/togosite/variation/annotation/clinvar>
FROM <http://rdf.integbio.jp/dataset/togosite/clinvar>
WHERE {  
{{#if categoryArray}}
  VALUES ?category { {{#each categoryArray}} "{{this}}" {{/each}} }    
{{/if}}
  ?togovar dct:identifier ?tgv_id.
  ?togovar rdfs:seeAlso ?label.
  ?togovar tgvo:condition/rdfs:seeAlso/cvo:interpreted_record/cvo:rcv_list/cvo:rcv_accession/cvo:interpretation ?category.  
}
GROUP BY ?id ?parent ?label
```

## `return`
- 整形
```javascript
({data})=>{
  const idVarName = "tgv_id";
  const idPrfix = "";
  const categoryPrefix = "";
  return data.results.bindings.map(d=>{
    return {
      id: d[idVarName].value.replace(idPrfix, ""), 
      attribute: {
        categoryId: d.category.value.toLowerCase().replace("/", "_or_").replace(/,?\s+/g, "_"),
        uri: d.category.value.toLowerCase().replace("/", "_or_").replace(/,?\s+/g, "_"),
        label : d.label.value.charAt(0).toUpperCase() + d.label.value.slice(1)   // 先頭の１文字だけを大文字にする。
      }
    }
  });
}
```