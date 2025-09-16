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
