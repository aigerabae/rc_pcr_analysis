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
```bash
conda create -p /workflow/envs/shankey_env plotly=5.18.0 pandas pip -c conda-forge
conda activate /workflow/envs/shankey_env
pip install kaleido==0.2.1
```

Now it works (with pointing to created environment)

Next error - cant finish 7b because it doesn't see the environment. so i need to point it to another enivornment or chnage it
I made it use the overall environment in step 7b:
```nf
    conda '/miniconda/envs/WGS_COVID19'
```

Then it lacked permissions to use scripts so inside that folder 9not in docker or environent):
```bash
chmod +x scripts/**
```

Then it lacked module weasyprint so i installed inside docker and that environment
```bash
pip install weasyprint==53.3
```
It also lacked jinja2
```bash
pip install jinja2
```

Then it had old versions of pango and cairo so I tried updatying them directly in the docker image:
```bash
apt-get update && apt-get install -y libcairo2 libpango-1.0-0 libgdk-pixbuf2.0-0
```

Then it lacked pandas and plotly and kaleido (needs version <1 to avoid having to install google chrome)
```bash
pip install pandas
pip install plotly
pip install kaleido==0.2.1
pip install matplotlib
pip install weasyprint
```

Needs some dependencies for weasyprint:
```bash
apt-get update && apt-get install -y \
  libcairo2 \
  libpango-1.0-0 \
  libpangocairo-1.0-0 \
  libpangoft2-1.0-0 \
  libgdk-pixbuf2.0-0 \
  libharfbuzz0b \
  libfreetype6 \
  libfontconfig1 \
  libffi-dev
```

It asked me again to install jinja2
```bash
pip install jinja2
```

Might need to use a different container because this one uses Ubuntu 14 which very dated. But the reason why I decided to use that container is because I needed an old enough Nextflow to use DSL1 with it. Might want to install older nextflow using older releases on githb: https://github.com/nextflow-io/nextflow/releases/tag/v22.10.0 Problem again is that nextflow 22.10.0 requires java8 to java18 while i have java24

I tried to create a docker image but now when i try to build it i get this error:
docker build -f ./Dockerfile .
[+] Building 0.0s (1/1) FINISHED                                                                             docker:default
 => [internal] load build definition from Dockerfile                                                                   0.0s
 => => transferring dockerfile: 2B                                                                                     0.0s
ERROR: failed to solve: failed to read dockerfile: open Dockerfile: no such file or directory
(base) aygera@aygera-HP-Z6-G4-Workstation:~/biostar/NCB/for_container$ 

Trying it in a different docker image (nextflowcent):
```bash
sh docker/run.sh try2 jonovox/nextflowcentos:latest
```

Inside container:
```bash
pip install weasyprint
pip install jinja2
conda install -c conda-forge "pango>=1.44"
nextflow run RC-PCR.nf   --reads '/workflow/*_R{1,2}_001.fastq.gz'   --outDir '/workflow/output'   --database SILVA   --threads 8   -profile docker   -resume
```

Conda install was loading for too long so i decided to try making a container myself again. in NCB folder:
```bash
docker pull ubuntu:22.04
docker run -it \
  -v $(pwd):/workspace \
  -w /workspace \
  ubuntu:22.04 \
  bash
apt-get update
apt-get install -y libasound2 libcups2 libx11-6 libxext6 libxrender1 libxtst6 libxi6   # needed for Oracle JDK .deb
apt-get remove -y jdk-18
apt --fix-broken install -y
apt-get install -y openjdk-17-jdk
java -version
```
