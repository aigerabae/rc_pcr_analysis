starting again. using v1,v6-v9 files and run batch script; also using nf_changed. 

docker pull jonovox/nextflowcentos:latest
downloading and unpacking https://surfdrive.surf.nl/files/index.php/s/5q2feFVult4v81k
copying all folders and files from rc_pcr folder

opening container (not necesary, dont do it to run batch):
sh docker/run.sh RC jonovox/nextflowcentos:latest

outside it running batch after putting files into workflow folder:
bash run_batch_docker.sh /home/aygera/biostar/NCB/attempt2/input/ _001.fastq.gz SILVA 8 jonovox/easyseq_covid19:latest SILVA_test
