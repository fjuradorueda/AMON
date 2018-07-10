# MicroMetabPred
A command line tool for predicting the compounds produced by microbes and the host.

## Installation
It is recommended to install MicroMetabPred in a conda environment. The environment can be created by first downloading the environment file.
```bash
wget https://raw.githubusercontent.com/shafferm/MicroMetabPred/master/environment.yaml
```

Then create a new conda environment. Using the environment file and activate it.
```bash
conda env create -f environment.yaml -n microMetabPred
conda activate microMetabPred
```

Then it can be installed via pip.
```bash
pip install MicroMetabPred
```

### Alternative installation
Alternatively microMetabPred can be installed from pip directly.
```bash
pip install MicroMetabPred
```

## Running MicroMetabPred
MicroMetabPred includes two scripts. `extract_ko_genome_from_organism.py` takes a KEGG organism flat file and makes a list of KOs present in that file. `MicroMetabPred.py` predicts the metabolites that could be produced by the KOs used as input. This can be compared to the KOs present in the host or from some other gene set as well as to as set of KEGG metabolites.

### `extract_ko_genome_from_organism.py`
A simple script. Takes a download of an organism file from KEGG and outputs a new line separate list of KOs present in that file.
```
extract_ko_genome_from_organism.py --help
usage: extract_ko_genome_from_organism.py [-h] -i INPUT -o OUTPUT

optional arguments:
  -h, --help            show this help message and exit
  -i INPUT, --input INPUT
                        KEGG organism flat file (default: None)
  -o OUTPUT, --output OUTPUT
                        Output file of new line separated list of KOs from
                        genome (default: None)
```

### `MicroMetabPred.py`
The full script to preform an analysis of possible metabolites originating from the list of KOs. From this as well as optional lists of compounds detected via metabolomics and lists of KOs present in a host or other environment a table of possible origin of compounds can be generated. From the list of compounds that could possibly be generated a pathway enrichment is also done with the hypergeometric test. Also if either of the other lists are included a Venn diagram will be generated representing the compounds which can be produced or where measured between the lists. If both the bacterial and host KOs are given a heatmap of pathway enrichments will be generated as well and in the enrichment test only compounds which are predicted to be uniquely generated by the bacteria or the host will be used.

#### Inputs

The `input` parameter is a list that can be in the form of a plain text file that is a white space separated list of KO ids, a tsv or csv where the column labels are KO ids or a biom formatted file where the observation ids are KO ids. These are the KOs that will be used to determine the compounds that could be generated by the bacterial community. This and the output directory where all results will be written are the only required requirements. There are two other optional inputs: `detected_compounds` and `host_kos`. `detected_compounds` is a set of compounds that where detected in metabolomics of the sample and can come in any of the forms available for the input. `host_kos` is a set of KO ids that are encoded by the host or another set of genes that can be expressed as KO ids. This can also take any of the forms available to the  input parameter.

Two flags are available that will affect the Venn diagram made and the enrichment analysis that is done. `detected_only` will only include compounds that were detected as the background set of compounds for the hypergeometric test. This flag requires the `compound_detected` variable to be used. The `rn_compound_only` flag makes it so that only detected compounds which have a reaction associated with them in KEGG will be used for both the Venn diagram and the hypergeometric test.

Finally a set of locations for KEGG FTP downloaded files is avaliable. These inputs are optional and if they are not provided the KEGG API will be used to retrieve the records necessary. It is much faster to run with the KEGG FTP downloaded files if you have access to them.

**NOTE: the KEGG API has limits. It is currently past the limits of the KEGG API to require all inputs to be pulled from the KEGG API with a reasonably sized data set. This is something I am working on and if you have any suggestions for how to work within these limits please create an issue or pull request with a fix.**

#### Outputs

All outputs are written to the `output` directory. If only the `input` parameter is given then two files will be generated called origin_table.tsv and bacteria_enrichment.tsv. The origin_table.tsv has rows as the compounds that could be generated and the first column is true or false indicating if the bacterial KOs provided could generate this KO. If the `host_kos` input is provided an additional column will be generated in this table with true/false values indicating if this set of KOs could generate these compounds. If the `detected_compounds` parameter is given then an additional column with true/false values indicating whether or not this compound was generated is added.

The bacteria_enrichment.tsv file, and the host_enrichment.tsv file if the `host_kos` parameter is given, gives the results of the pathway enrichment analysis from the compounds able to be produced by the KOs provided. When the `host_kos` parameter is given a heatmap is made to compare the significant pathways present from the bacteria and host KO lists.

When the `host_kos` and/or `detected_compounds` parameters are given a venn diagram will be made to see overlap in compounds possibly generated or detected.

#### Full help
```
MicroMetabPred.py --help
usage: MicroMetabPred.py [-h] -i INPUT -o OUTPUT_DIR
                         [--detected_compounds DETECTED_COMPOUNDS]
                         [--host_kos HOST_KOS] [--detected_only]
                         [--rn_compound_only] [--ko_file_loc KO_FILE_LOC]
                         [--rn_file_loc RN_FILE_LOC]
                         [--co_file_loc CO_FILE_LOC]
                         [--pathway_file_loc PATHWAY_FILE_LOC]

optional arguments:
  -h, --help            show this help message and exit
  -i INPUT, --input INPUT
                        KEGG KO's from bacterial community in the form of a
                        white space separated list, a tsv or csv with KO ids
                        as column names or a biom file with KO ids as
                        observations (default: None)
  -o OUTPUT_DIR, --output_dir OUTPUT_DIR
                        directory to store output (default: None)
  --detected_compounds DETECTED_COMPOUNDS
                        list of compounds detected via metabolomics (default:
                        None)
  --host_kos HOST_KOS   white space separated list of KEGG KO's from host or
                        other environment (default: None)
  --detected_only       only use detected compounds in enrichment analysis
                        (default: False)
  --rn_compound_only    only use compounds with associated reactions (default:
                        False)
  --ko_file_loc KO_FILE_LOC
                        Location of ko file from KEGG FTP download (default:
                        None)
  --rn_file_loc RN_FILE_LOC
                        Location of reaction file from KEGG FTP download
                        (default: None)
  --co_file_loc CO_FILE_LOC
                        Location of compound file from KEGG FTP download
                        (default: None)
  --pathway_file_loc PATHWAY_FILE_LOC
                        Location of pathway file from KEGG FTP download
                        (default: None)
```
