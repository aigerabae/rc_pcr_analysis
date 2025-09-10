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

Single batch:
```bash
sh docker/run.sh RC jonovox/nextflowcentos:latest
# opens docker 
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
