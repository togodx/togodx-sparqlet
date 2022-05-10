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
    'organism_01': 'Homo sapiens (human)',
    'organism_02': 'Pan troglodytes (chimpanzee)',
    'organism_03': 'Macaca mulatta (Rhesus monkey)',
    'organism_04': 'Mus musculus (mouse)',
    'organism_05': 'Rattus norvegicus (rat)',
    'organism_06': 'Canis lupus familiaris (dog)',
    'organism_07': 'Bos taurus (cow)',
    'organism_08': 'Gallus gallus (chicken)',
    'organism_09': 'Xenopus tropicalis (western clawed frog)',
    'organism_10': 'Danio rerio (zebrafish)',
    'organism_11': 'Drosophila melanogaster (fruit fly)',
    'organism_12': 'Anopheles gambiae (malaria mosquito)',
    'organism_13': 'Caenorhabditis elegans (nematode)',
    'organism_14': 'Saccharomyces cerevisiae (budding yeast)',
    'organism_15': 'Schizosaccharomyces pombe (fission yeast)',
    'organism_16': 'Kluyveromyces lactis (ascomycetes)',
    'organism_17': 'Eremothecium gossypii (ascomycetes)',
    'organism_18': 'Magnaporthe oryzae (rice blast fungus)',
    'organism_19': 'Neurospora crassa (ascomycetes)',
    'organism_20': 'Arabidopsis thaliana (thale cress)',
    'organism_21': 'Oryza sativa (rice)',
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