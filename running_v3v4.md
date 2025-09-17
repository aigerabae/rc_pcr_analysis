# docker run -it jonovox/rcpcr_ready_image 
# to open it if you need to install something from inside
bash run_batch_docker.sh /home/aygera/biostar/NCB/attempt2/input_v3v4/ _001.fastq.gz ILLUMINA_V3V4 24 rcpcr_ready_image:latest output_v3_v4

First step - creating database for primers.
In attempt2 folder:
cd db/
mkdir ILLUMINA_V3V4
cd ILLUMINA_V3V4/
mkdir primers

```amplicons_16S
#V3-V4 
#16S Amplicon PCR Forward Primer
#FowrdPrim	ReversePrim	InserLength	AuxInfo
CCTACGGGNGGCWGCAG GACTACHVGGGTATCTAATCC 460 primers

# I used 460 as average. might want to recalculate
# I used those as i removed overhangs from themz
```

```ILLUMINA_V3V4_primers
>primers_F
CCTACGGGNGGCWGCAG
>primers_R
GACTACHVGGGTATCTAATCC
```

I also copied KMA folder to db/ILLUMINA_V3V4 and renamed all SILVA files to ILLUMINA_V3V4

I modified shankey.py script to say unclassified if data are unavailable:
```shankey.py
def createdf(input):
    df = pd.read_csv(input, sep="\t")

    print(df["#Template"])

    df = df[df["Depth"] >= 1]
    df = df[df["fragmentCount"] >= 100]
    df = df[df["Template_Coverage"] >= 50]

    lvls = df["#Template"].str.split(";", expand = True)
    
    df["lvl1"] = "Bacteria"
    
    if len(lvls) > 1:
        df["lvl2"] = lvls[1]
        df["lvl3"] = lvls[2]
        df["lvl4"] = lvls[3]
        df["lvl5"] = lvls[4]
        df["lvl6"] = lvls[5]
        df["lvl7"] = lvls[6]
    else:
        df["lvl2"] = "Unclassified"
        df["lvl3"] = "Unclassified"
        df["lvl4"] = "Unclassified"
        df["lvl5"] = "Unclassified"
        df["lvl6"] = "Unclassified"
        df["lvl7"] = "Unclassified"
    return df
```

This results in an empty shankey plot. I will test this code to see if it works for v1-v6,v9:
conda activate /home/aygera/biostar/NCB/attempt2/conda/env-a1b65765c2bf2d3f21605d8abc9f0cd9
cd /home/aygera/biostar/NCB/attempt2/test_results/
python test_shankey.py --res ../output_output_v3_v4/MetaG-Ayan-A3/kma/MetaG-Ayan-A3.res
python test_shankey.py --res ../output_output_v1v6v9/MetaGV9-A3/kma/MetaGV9-A3.res

Chercking properties:
import pandas as pd
df = pd.read_csv("../output_output_v3_v4/MetaG-Ayan-A3/kma/MetaG-Ayan-A3.res", sep="\t")
print(df['Depth'].describe())
print(df['Depth'].unique()[:10])
print(df.columns)
lvls = df["#Template"].str.split(";", expand = True)
df["lvl1"] = "Bacteria"
df["lvl2"] = lvls[1]
df["lvl3"] = lvls[2]
df["lvl4"] = lvls[3]
df["lvl5"] = lvls[4]
df["lvl6"] = lvls[5]
df["lvl7"] = lvls[6]
print(df[['lvl1','lvl2','lvl3','lvl4','lvl5','lvl6','lvl7']].head(10))

Alright. I now know the issue. There is filtering and it removed everything. so the plot is empty. now i want to make interactive html plot like qiime

Inputs:
  --i-table ARTIFACT FeatureTable[Frequency | PresenceAbsence]
                         Feature table to visualize at various taxonomic
                         levels.                                    [required]
  --i-taxonomy ARTIFACT FeatureData[Taxonomy]
                         Taxonomic annotations for features in the provided
                         feature table. All features in the feature table must
                         have a corresponding taxonomic annotation. Taxonomic
                         annotations that are not present in the feature table
                         will be ignored. If no taxonomy is provided, the
                         feature IDs will be used as labels.        [optional]
Parameters:
  --m-metadata-file METADATA...
    (multiple            The sample metadata.
     arguments will be
     merged)                                                        [optional]
  --p-level-delimiter TEXT
                         Attempt to parse hierarchical taxonomic information
                         from feature IDs by separating levels with this
                         character. This parameter is ignored if a taxonomy is
                         provided as input.                         [optional]
Outputs:
  --o-visualization VISUALIZATION
                                                                    [required]
Miscellaneous:
  --output-dir PATH      Output unspecified results to a directory
  --verbose / --quiet    Display verbose output to stdout and/or stderr
                         during execution of this action. Or silence output if
                         execution is successful (silence is golden).
  --example-data PATH    Write example data and exit.
  --citations            Show citations and exit.
  --use-cache DIRECTORY  Specify the cache to be used for the intermediate
                         work of this action. If not provided, the default
                         cache under $TMP/qiime2/ will be used.
                         IMPORTANT FOR HPC USERS: If you are on an HPC system
                         and are using parallel execution it is important to
                         set this to a location that is globally accessible to
                         all nodes in the cluster.
  --help                 Show this message and exit.

Examples:
  # ### example: barplot
  qiime taxa barplot \
    --i-table table.qza \
    --i-taxonomy taxonomy.qza \
    --m-metadata-file sample-metadata.tsv \
    --o-visualization taxa-bar-plots.qzv

python test_shankey.py --res ../output_output_v3_v4/MetaG-Ayan-A3/kma/MetaG-Ayan-A3.res

Installing with conda:
conda env create -n qiime2-metagenome-2024.10 --file https://data.qiime2.org/distro/metagenome/qiime2-metagenome-2024.10-py310-linux-conda.yml

Now need to run:
./kma_to_qiime.py --res-files ../output_output_v1v6v9/MetaGV9-A3/kma/MetaGV9-A3.res ../output_output_v1v6v9/MetaGV9-C7/kma/MetaGV9-C7.res -o qiime_out
qiime tools import \
  --input-path qiime_out/feature-table.tsv \
  --type 'FeatureTable[Frequency]' \
  --input-format BIOMV100Format \
  --output-path qiime_out/table.qza

qiime tools import \
  --input-path qiime_out/taxonomy.tsv \
  --type 'FeatureData[Taxonomy]' \
  --output-path qiime_outqiime_outtaxonomy.qza

qiime taxa barplot \
  --i-table qiime_outtable.qza \
  --i-taxonomy qiime_outtaxonomy.qza \
  --o-visualization qiime_outtaxa-bar-plots.qzv
