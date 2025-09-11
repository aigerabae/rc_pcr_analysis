Installign docker:
```bash
sudo snap install docker
# solving issue with user not allowed
sudo groupadd docker
sudo usermod -aG docker $USER
```

Installing nextflow:
```bash
wget https://get.nextflow.io | bash
chmod +x nextflow
mkdir -p $HOME/.local/bin/
mv nextflow $HOME/.local/bin/
export PATH="$PATH:$HOME/.local/bin"
~/.bashrc
nextflow info
```

Downloading docker image:
```bash
docker pull jonovox/nextflowcentos:latest
conda env create -f 16S_ID.yaml
```

Single run:
```bash
sh docker/run.sh RC jonovox/nextflowcentos:latest
# opens docker; to exit type exit
# RC - container name
# jonovox/nextflowcentos:latest - This is the Docker image you’re asking Docker to run
```

This is run.sh file:
```bash
# ${1} <runtimecontainername> example: RC
# ${2} <imagename> example: jonovox/nextflowcentos:latest 
docker run \
-it \
--rm \
--name ${1} \
--mount type=bind,source=${PWD},target=/workflow \
```

Batch running (their example):
```bash
# USAGE:
# bash run_batch_docker.sh <inputpath> <file_extension> <database> <threads> <image> <outputname>
# bash run_batch_docker.sh ${1}             ${2}          ${3}        ${4}     ${5}   ${6}
# Example:
bash scripts/run_batch_docker.sh /media/aygera/external_disk/biostar/NCB/RC-PCR/test _001.fastq.gz SILVA 8 jonovox/easyseq_covid19:latest SILVA_test
```
I wasn't able to run this because it gave this error at some point so i stopped it. I deleted all packages from my conda installation and then realized it wasn't necesary and decided to use tarball conda but didn't succeed in installing. I then installed nextflow via conda which also didn't work that well (no errors but im not sure if it installed even)

scripts/run_batch_docker.sh:
```bash
#!/bin/bash

# USAGE
# bash run_batch_docker.sh <inputpath> <file_extension> <database> <threads> <image> <outputname>
# bash run_batch_docker.sh ${1}             ${2}          ${3}        ${4}     ${5}   ${6}
# <file_extension> most common _001.fastq.gz

for fname in ${1}/*_R1${2}
do
    base=${fname##*/}
    base=${base%_R1*}
    echo "${base}_R1${2}"
    echo "${base}_R2${2}"

    docker run -it --rm --mount type=bind,source=${PWD},target=/workflow \
    --mount type=bind,source=${1},target=/workflow/input \
    ${5} nextflow run RC-PCR.nf \
    --reads "/workflow/input/${base}_R{1,2}${2}" --outDir /workflow/output_${6}/ \
    --threads ${4} --database ${3} -resume --UMILEN 25 --abricate false -with-dag flowchart.png
done
```

Running my own data. I decided to use docker image with nextflow 20 (that supports previous vesion of nextflow script that i have). I opened v3-v4 directory and inside it ran:
```bash
sh /media/aygera/external_disk/biostar/NCB/RC-PCR/docker/run.sh RC jonovox/easyseq_covid19:latest
```

Then it opened container and inside it i ran:
```bash
nextflow run RC-PCR.nf \
  --reads '/workflow/*_R{1,2}_001.fastq.gz' \
  --outDir '/workflow/output' \
  --database SILVA \
  --threads 8 \
  -profile docker
```

Adding something to directory then continuing:
```bash
nextflow run RC-PCR.nf \
  --reads '/workflow/*_R{1,2}_001.fastq.gz' \
  --outDir '/workflow/output' \
  --database SILVA \
  --threads 8 \
  -profile docker \
  -resume
```

Current script needs to have in the directiry with data also:
- bin with certain scripts
- conda with conda packages
- db with SILVA primers
- output folder for output
- scripts folder with certain scripts
- work folder for temporary files

I manually downloaded mkl-2023.1.0-h213fc3f_46344.tar.bz2 package from conda and copied it into docker miniconda/packaGES:
```bash
docker cp ~/mkl-2023.1.0-h213fc3f_46344.tar.bz2 4d5783f9e118:/miniconda/pkgs/
```

Then the next error was about python script shankey.py. The problem was that it didn't work because it filtered out all values in .res file. So i temporarily removed filtering to see if other steps would work. (commented filtering steps, will uncomment later)
```python
def createdf(input):
    df = pd.read_csv(input, sep="\t")

    print(df["#Template"])

    #df = df[df["Depth"] >= 1]
    #df = df[df["fragmentCount"] >= 100]
    #df = df[df["Template_Coverage"] >= 50]

    lvls = df["#Template"].str.split(";", expand = True)

    df["lvl1"] = "Bacteria"
    df["lvl2"] = lvls[1]
    df["lvl3"] = lvls[2]
    df["lvl4"] = lvls[3]
    df["lvl5"] = lvls[4]
    df["lvl6"] = lvls[5]
    df["lvl7"] = lvls[6]

    return df
```

Next error was about plotly and kaleidos versions, so inside the container I ran:
```bash
pip install kaleido==0.2.1
```

It still didn't solve it so i edited the .nf script to contain specific versions (original version commented out but in the file i had to remove the commented line because it created a parsing error):
```script
// Visualization using shankeyplot
process '5B_shankey_viz' {
    tag '5B'
    #conda 'plotly::plotly anaconda::pandas conda-forge::python-kaleido'
    conda 'conda-forge::plotly=5.18.0 conda-forge::kaleido=0.2.1 conda-forge::pandas'
    publishDir outDir + '/report/viz', mode: 'copy'
    input:
        file res from kma_5B
    output:
        file ".command.*"
        file "shankey.png" into shankey_7B
    script:
        """
        python ${baseDir}/bin/shankey.py --res ${res}
        """
}
```

This also resulted in the next error that was due to unstable internet and old conda. Thus I created env_shankey.yml in workflow directory:
```bash
name: shankey_env
channels:
  - conda-forge
dependencies:
  - plotly=5.18.0
  - pandas
  - pip
  - pip:
      - kaleido==0.2.1
```

Then created the environment:
```bash
conda env create -f env_shankey.yml -p /workflow/envs/shankey_env
```

Then edited .nf file to point it to that environment:
```nf
// Visualization using shankeyplot
process '5B_shankey_viz' {
    tag '5B'
    conda '/workflow/envs/shankey_env'
    publishDir outDir + '/report/viz', mode: 'copy'
    input:
        file res from kma_5B
    output:
        file ".command.*"
        file "shankey.png" into shankey_7B
    script:
        """
        python ${baseDir}/bin/shankey.py --res ${res}
        """
}
```

This still didn't work so I tried creatign the environment manually and then installling it via pip after activating:
conda create -p /workflow/envs/shankey_env plotly=5.18.0 pandas pip -c conda-forge
conda activate /workflow/envs/shankey_env
pip install kaleido==0.2.1
