# uniprot keywords “ñ€ŠÖŒWiç‰®j

* UniProt keyword ‚Ì“à–ó
  * ?uniprot up:classifiedWith [keyword ID]
  * [keyword ID] rdfs:subClassOF+ [10 keyword types]
    * 9990: Technical term (1ŠK‘w (ŠK‘w‚È‚µ))
    * 9991: PTM (Å‘å3ŠK‘w)
    * 9992: Molecular function (Å‘å5ŠK‘w)
    * 9993: Ligand (Å‘å5ŠK‘w)
    * 9994: Domain (Å‘å2ŠK‘w)
    * 9995: Disease (Å‘å3ŠK‘w)
    * (9996: Developmental stage (Å‘å3ŠK‘w) human –³‚µ)
    * 9997: Coding sequence diversity (1ŠK‘w (ŠK‘w‚È‚µ))
    * 9998: Cellular component (Å‘å4ŠK‘w)
    * 9999: Biological process (Å‘å8ŠK‘w)

## Parameters

* `root` (type: UniProt keyword ID) (Req.)
  * default: 9995

## Endpoint
https://integbio.jp/togosite/sparql

## `data`
- Attribute‚ÌŠK‘wŠÖŒWAAttribute‚Æ—v‘fiUniProtj‚ÌŠÖŒW‚ğ“ñ€ŠÖŒW‚Åæ“¾
```sparql
PREFIX up: <http://purl.uniprot.org/core/>
PREFIX taxon: <http://purl.uniprot.org/taxonomy/>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX skos: <http://www.w3.org/2004/02/skos/core#>
PREFIX uniprot: <http://purl.uniprot.org/uniprot/>
PREFIX keywords: <http://purl.uniprot.org/keywords/>
SELECT DISTINCT ?parent ?child ?parent_label ?child_label ?leaf
FROM <http://rdf.integbio.jp/dataset/togosite/uniprot>
FROM <http://rdf.integbio.jp/dataset/togosite/uniprot/keywords>
WHERE {
  VALUES ?root { keywords:{{root}} }
  {
    ?child a up:Protein ;
           up:organism taxon:9606 ;
           up:proteome ?proteome ;
           up:classifiedWith ?parent .
    FILTER(REGEX(STR(?proteome), "UP000005640"))
    ?parent a up:Concept ;
            rdfs:subClassOf* ?root .
    ?child up:mnemonic ?child_label .
    BIND(1 AS ?leaf)
  } UNION {
    ?child rdfs:subClassOf* ?root ;
           a up:Concept ;
           rdfs:subClassOf ?parent .
    ?parent rdfs:subClassOf* ?root . # keywords ‚Í•¡”‚ÌƒJƒeƒSƒŠ‚É‚Ô‚ç‰º‚ª‚é‚±‚Æ‚ª‚ ‚é‚Ì‚Åe‚Ìroot‚àƒ`ƒFƒbƒN
    ?child skos:prefLabel ?child_label .
    BIND(0 AS ?leaf)
  }
  ?parent skos:prefLabel ?parent_label .
}
```

## `return`
- ®Œ`
```javascript
({root, data})=>{
  const idPrefix = "http://purl.uniprot.org/uniprot/";
  const categoryPrefix = "http://purl.uniprot.org/keywords/";

  let tree = [{
    id: root,
    root: true
  }];
  
  data.results.bindings.map(d => {
    tree.push({
      id: d.child.value.replace(categoryPrefix, "").replace(idPrefix, ""),
      label: d.child_label.value,
      leaf: Boolean(Number(d.leaf.value)),
      parent: d.parent.value.replace(categoryPrefix, "")
    })
    if (d.parent.value.replace(categoryPrefix, "") == root && !tree[0].label) tree[0].label = d.parent_label.value;
  });
  
  return tree;
}
```