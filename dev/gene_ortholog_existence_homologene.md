# ヒト遺伝子が保存されている生物（千葉）

## Description
- Label
  - Ortholog existence
- Description
  - Ortholog existence for each human gene in other organisms defined by HomoloGene
- Order  
  - id_asc
- Source
  - [HomoloGene](https://www.ncbi.nlm.nih.gov/homologene/statistics/) : https://www.ncbi.nlm.nih.gov/homologene/statistics/
- Version
  - 68
- Updated
  - 2021-03-12
- Key
  - NCBI gene

## Endpoint

https://integbio.jp/togosite/sparql


## `main`

```sparql
PREFIX orth: <http://purl.org/net/orth#>
PREFIX homologene: <https://ncbi.nlm.nih.gov/homologene/>
PREFIX taxid: <http://identifiers.org/taxonomy/>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>

SELECT ?human_gene ?gene_label ?taxid
FROM <http://rdf.integbio.jp/dataset/togosite/homologene/ontology>
FROM <http://rdf.integbio.jp/dataset/togosite/homologene/data>
WHERE {
  ?grp orth:inDataset homologene: ;
      orth:hasHomologousMember ?human_gene, ?gene .
  ?human_gene orth:taxon taxid:9606 ;
      rdfs:label ?gene_label .
  ?gene orth:taxon ?taxid .
}
```

## `return`

```javascript
({ main }) => {
   
  let tree = [
    {
      id: 'root',
      label: 'root node',
      root: true
    }
  ];

  const taxidMap = {
    '9606': 'organism_01',
    '9598': 'organism_02',
    '9544': 'organism_03',
    '10090': 'organism_04',
    '10116': 'organism_05',
    '9615': 'organism_06',
    '9913': 'organism_07',
    '9031': 'organism_08',
    '8364': 'organism_09',
    '7955': 'organism_10',
    '7227': 'organism_11',
    '7165': 'organism_12',
    '6239': 'organism_13',
    '4932': 'organism_14',
    '4896': 'organism_15',
    '28985': 'organism_16',
    '33169': 'organism_17',
    '318829': 'organism_18',
    '5141': 'organism_19',
    '3702': 'organism_20',
    '4530': 'organism_21',
  };

  const organismLabel = {
    'organism_01': 'human (Homo sapiens)',
    'organism_02': 'chimpanzee (Pan troglodytes)',
    'organism_03': 'Rhesus monkey (Macaca mulatta)',
    'organism_04': 'mouse (Mus musculus)',
    'organism_05': 'rat (Rattus norvegicus)',
    'organism_06': 'dog (Canis lupus familiaris)',
    'organism_07': 'cow (Bos taurus)',
    'organism_08': 'chicken (Gallus gallus)',
    'organism_09': 'western clawed frog (Xenopus tropicalis)',
    'organism_10': 'zebrafish (Danio rerio)',
    'organism_11': 'fruit fly (Drosophila melanogaster)',
    'organism_12': 'malaria mosquito (Anopheles gambiae)',
    'organism_13': 'nematode (Caenorhabditis elegans)',
    'organism_14': 'budding yeast (Saccharomyces cerevisiae)',
    'organism_15': 'fission yeast (Schizosaccharomyces pombe)',
    'organism_16': 'ascomycetes (Kluyveromyces lactis)',
    'organism_17': 'ascomycetes (Eremothecium gossypii)',
    'organism_18': 'rice blast fungus (Magnaporthe oryzae)',
    'organism_19': 'ascomycetes (Neurospora crassa)',
    'organism_20': 'thale cress (Arabidopsis thaliana)',
    'organism_21': 'rice (Oryza sativa)',
  }

  let checked = {};

  main.results.bindings.forEach((elem) => {
    const taxid = elem.taxid.value.replace('http://identifiers.org/taxonomy/', '');
    const organismId = taxidMap[taxid];
    if (!checked[taxid]) {
      checked[taxid] = true;
      tree.push({
        id: organismId,
        label: organismLabel[organismId],
        parent: 'root',
      });
    }
    tree.push({
      id: elem.human_gene.value.replace('http://identifiers.org/ncbigene/', ''),
      label: elem.gene_label.value,
      leaf: true,
      parent: taxidMap[taxid]
    });
  });

  return tree;
};
```
