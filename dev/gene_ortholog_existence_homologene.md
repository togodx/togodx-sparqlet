# ヒト遺伝子が保存されている生物（千葉）

## Description
- Name
  - Ortholog existence
- Description
  - Ortholog existence for each human gene in other organisms in HomoloGene
- Original data
  - [HomoloGene](https://www.ncbi.nlm.nih.gov/homologene/statistics/)
- Version
  - 68

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
    'organism_01': 'Human',
    'organism_02': 'Chimpanzee',
    'organism_03': 'Rhesus monkey',
    'organism_04': 'Mouse',
    'organism_05': 'Rat',
    'organism_06': 'Dog',
    'organism_07': 'Cow',
    'organism_08': 'Chicken',
    'organism_09': 'Western clawed frog',
    'organism_10': 'Zebrafish',
    'organism_11': 'Fruit fly',
    'organism_12': 'Malaria mosquito',
    'organism_13': 'Nematode',
    'organism_14': 'Budding yeast',
    'organism_15': 'Fission yeast',
    'organism_16': 'Kluyveromyces lactis',
    'organism_17': 'Eremothecium gossypii',
    'organism_18': 'Magnaporthe oryzae',
    'organism_19': 'Neurospora crassa',
    'organism_20': 'Arabidopsis thaliana',
    'organism_21': 'Oryza sativa',
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