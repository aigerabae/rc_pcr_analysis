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
