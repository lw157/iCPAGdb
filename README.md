# iCPAGdb -- LD-aware SNP overlap across multiple GWAS datasets

Web browser: http://cpag.oit.duke.edu/ 

This repo contains both back-end and front-end codes of iCPAGdb and the web browser.

iCPAGdb integrates the results of GWAS across different phenotypic scales, identifying and quantifying the significance of pleiotropic loci that impact molecular, cellular, and organismal traits. The goal is to provide a resource that allows experts on a particular human trait to easily develop hypotheses for molecular and cellular phenotypes that underlie the physiology of that trait. Molecules and cellular pathways implicated in this way could serve as novel biomarkers or targets for therapeutic approaches. The current version of iCPAGdb contains GWAS summary statistics from >4400 diseases/traits and allows users to explore pre-computed correlations across all existing diseases and/or upload their own GWAS to identify and explore shared SNPs between their own GWAS and >4400 diseases/traits.

This repo contains two parts
1) python (3.6+) code for iCPAGdb

2) R shiny code for Web browser


# Quick start

## Configuration and download database and third-party software
1) direct download [PLINK 1.9](https://www.cog-genomics.org/plink/1.9/), or using Linux/Max ```wget``` function  and place it to folder "plink_bins"  <br/>

```sh 
## please choose the proper PLINK version, here is an example of the Linux version 

wget http://s3.amazonaws.com/plink1-assets/plink_linux_x86_64_20201019.zip
```

2) download the ziped database file (~15Gb), and decompress it to "db" folder from Dropbox [LINK](https://www.dropbox.com/sh/na23jflxcgk0nib/AAAoOj3gB8k8j_dNH1UBFzeZa?dl=0). The following is an example of downloading commands on Linux/Mac using ```wget```.

```sh 
wget https://www.dropbox.com/sh/na23jflxcgk0nib/AAAoOj3gB8k8j_dNH1UBFzeZa\?dl=1  --content-disposition

```

The final folder structure contains all required codes and data file: <br/>
pyCPAGdb <br/>
├── _utils.py <br/>
├── anno_parent_efo.py <br/>
├── main.py <br/>
├── stats.py <br/>
├── plink_bins <br/>
│   ├── plink <br/>
│   └── prettify <br/>
├── db <br/>
│   ├── cpag_gwasumstat_v1.1.db <br/>
│   ├── cpag_gwasumstat_v1.1_AFR.ld0.4.db <br/>
│   ├── cpag_gwasumstat_v1.1_EAS.ld0.4.db <br/>
│   ├── cpag_gwasumstat_v1.1_EUR.ld0.4.db <br/>
│   ├── gwas-efo-trait-mappings.txt <br/>
│   └── lddat <br/>
│       ├── AFR_1kg_20130502_maf01.bed <br/>
│       ├── AFR_1kg_20130502_maf01.bim <br/>
│       ├── AFR_1kg_20130502_maf01.fam <br/>
│       ├── EAS_1kg_20130502_maf01.bed <br/>
│       ├── EAS_1kg_20130502_maf01.bim <br/>
│       ├── EAS_1kg_20130502_maf01.fam <br/>
│       ├── EUR_1kg_20130502_maf01.bed <br/>
│       ├── EUR_1kg_20130502_maf01.bim <br/>
│       └── EUR_1kg_20130502_maf01.fam <br/>

3) configure computing environment for python 3 

The fast way is to install [Miniconda](https://docs.conda.io/projects/conda/en/latest/user-guide/install/) and install required package from there.

Create a new environment using conda:

```sh
conda create -n icpagdb python=3.7
conda activate icpagdb
```

install python package using conda:

```sh
conda install -c conda-forge panda
conda install -c conda-forge scipy
conda install -c conda-forge joblib
conda install -c conda-forge tqdm
conda install -c conda-forge sqlite
```


## Run example (Shell command)

### Example 1

Serum metabolites/xenobiotics (Shin et al. 2014) vs. Human disease 

```sh
python main.py cpagdb --threads 2 --subtype NHGRI --NHGRI-Pcut 5e-8 \
  --subtype BloodMetabolites,BloodXenobiotic --Pcut 1e-5 \
  --lddb-pop EUR --outfile NHGRI-p1e-05-BloodMetabolitesXenobiotic-p1e-05-EUR.csv
```

then annotate phenotype:

```sh 
python main.py post_analysis --anno-ontology --anno-cols Trait1 \
  --infile output/NHGRI-p1e-05-BloodMetabolitesXenobiotic-p1e-05-EUR.csv \
  --outfile NHGRI-p1e-05-BloodMetabolitesXenobiotic-p1e-05-EUR.csv
```

### Example 2

```sh 
python main.py cpagdb --threads 2 --subtype H2P2 --H2P2-Pcut 1e-7 \
  --lddb-pop EUR --outfile output/H2P2-p1e-07-EUR.csv
```

### Example 3 (user GWAS)

download COVID-19 GWAS example from "Upload and compute CPAG" page at [HERE](http://cpag.oit.duke.edu/)

```sh 
python main.py usr-gwas --threads 10 --infile iCPAGdb-Sample-GWAS-top_EllinghausPCs_covid19.csv \
  --SNPcol "avsnp150" --delimitor "," --Pcol "p_value" \
  --usr-pcut 1e-5 \
  --outfile top_EllinghausPCs_covid19_pcut1e-5_icpagdb_out.csv
  ```

then annotate phenotype:

```sh 
python main.py post_analysis --anno-ontology --anno-cols Trait2 \
  --infile top_EllinghausPCs_covid19_pcut1e-5_icpagdb_out.csv \
  --outfile top_EllinghausPCs_covid19_pcut1e-5_icpagdb_out_addEFO.csv
  ```
### Example 4 (user multiple GWAS)

Compare user multiple GWAS files with LD-aware SNP sharing with a single p-value cutoff

```sh 
python main.py usr-multi-gwas --threads 10 --infile test_gwas1.txt test_gwas2.txt test_gwas3.txt \
  --SNPcol "SNP" --delimitor "," --Pcol "P" \
  --usr-pcut 1e-5 \
  --outfile test_output.csv
  ```


Compare user multiple GWAS files with LD-aware SNP overlap with multiple p-value cutoffs

```sh 
python main.py usr-multi-gwas --threads 10 --infile test_gwas1.txt test_gwas2.txt test_gwas3.txt \
  --SNPcol "SNP" --delimitor "," --Pcol "P" \
  --usr-pcut 1e-5 1e-6 1e-7 \
  --outfile test_output.csv
  ```


Compare user multiple GWAS files with LD-aware SNP overlaps, with different p-value cutoffs. In the following example, 1e-5 for test_gwas1.txt, and 1e-6 for test_gwas2.txt and test_gwas3.txt

```sh 
python main.py usr-multi-gwas --threads 10 --infile test_gwas1.txt test_gwas2.txt, test_gwas3.txt \
  --SNPcol "SNP" --delimitor "," --Pcol "P" \
  --usr-pcut 1e-5 1e-6 \
  --outfile test_output.csv
  ```
