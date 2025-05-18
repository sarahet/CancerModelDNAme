# CancerModelDNAme
Compare CGI methylation of primary tumors and cancer models

## Background
This repository contains an R markdown script to visualize CGI methylation of human and mouse samples based on the median methylation level and the fraction of methylated CGIs for three different CGI groups: PRC2 targets, hyper CGIs determined across a pan-cancer cohort and hyper CGIs determined for a specific tumor type. The script produces visualizations based on the reference created in Hetzel et al. 2025. Note: The human T-ALL samples used in this reference cannot be visualized. If you want to include the data, you need to apply via St. Jude to get access (EGA accession: EGAD00001007968) and include them as new samples. 

## Prerequisites

The script was tested using R version 4.2.2 but should work with any R version $\geq$ 4.0.0. The R package ```ggplot2``` needs to be installed.

The script allows you to visualize your own human or mouse samples together with the reference data. Before executing the script you need to generate average methylation values of CGIs for you new samples. Each sample should initially be in a separate ```bedgraph``` file that contains methylation rates per CpG, which has the following format:

```
<chr>  <start>  <end>  <methylation_rate>
```

An example could look like this:

```
chr1	10524	10526	1
chr1	10562	10564	0.95
chr1	10570	10572	0.941
...
```

Analogously to the reference, human (hg19) and mouse (mm10) samples can be provided and average methylation rates for each CGI can be easily calculated using ```bedtools``` and a CGI annotation that you can download for the respective reference genome from UCSC:

```
for F in /path/to/bedgraph/*.bed; do
    n=`basename $F | sed 's/.bed//'`
    intersectBed -a $F -b CGIs.bed -wa -wb | sort -k5,5 -k6,6n | groupBy -i stdin -g 5,6,7 -c 4,4 -o mean,count | awk '$5>2' OFS="\t" | cut -f 1-4 > /path/to/cgi_averages/avg_cgi_${n}.bed
done
```

The names of the output files should be ```avg_cgi_<name>.bed``` as in the example code above because the script uses this structure to infer sample names from the file. The output files will look like this:

```
chr1	28735	29810	0.01504742268
chr1	135124	135563	0.9737
chr1	327790	328229	0.9437894737
...
```

Additionally, you should prepare a ```csv``` file with information about your new samples similar to this example:

```
sample,type,condition,group,species
my_sample1,COAD,healthy,primary,human
my_sample2,COAD,tumor,primary,human
my_sample3,COAD,tumor,model,human
my_sample4,READ,tumor,model,human
```

1. The sample has to match the ```name``` within ```avg_cgi_<name>.bed``` so that the annotation and file can be mapped to each other.
2. The tumor type should be one of: ```B-ALL```, ```BLCA```, ```BRCA```, ```CESC```, ```CHOL```, ```COAD```, ```ESCA```, ```GBM```, ```HNSC```, ```KIRC```, ```KIRP```, ```LAML```, ```LIHC```, ```LUAD```, ```LUSC```, ```PAAD```, ```PCPG```, ```PRAD```, ```READ```, ```SARC```, ```SKCM```, ```STAD```, ```T-ALL```, ```THCA```, ```THYM``` or ```UCEC```. These are the types, which have pre-computed, type-specific hyper CGIs available. If your type does not match one of these, only the PRC2 target and the pan-cancer hyper set will be visualized.
3. The condition should be one of: ```healthy```, ```healthy_sorted```, ```precursor``` or ```tumor```.
4. The group should be ```primary``` or ```model```.
5. The species should be ```human``` or ```mouse```.
6. You can combine different species and tumor types within your new cohort.

## Usage

You can then run the script the following way:

```
Rscript -e "rmarkdown::render('visualize_cancer_model.Rmd',
params=list(
cgi_path = '/path/to/cgi_averages',
sample_path = '/path/to/sample_sheet.csv'),
output_file = 'my_output.pdf')"
```

The script needs to be executed from the CancerModelDNAme directory.
