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
