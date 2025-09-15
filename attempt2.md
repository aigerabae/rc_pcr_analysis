starting again. using v1,v6-v9 files and run batch script; also using nf_changed. 

```bash
docker pull jonovox/nextflowcentos:latest
```

downloading and unpacking https://surfdrive.surf.nl/files/index.php/s/5q2feFVult4v81k
copying all folders and files from rc_pcr folder

opening container (not necesary, dont do it to run batch):
```bash
sh docker/run.sh RC jonovox/nextflowcentos:latest
```

outside it running batch after putting files into workflow folder:
```bash
bash run_batch_docker.sh /home/aygera/biostar/NCB/attempt2/input/ _001.fastq.gz SILVA 8 jonovox/easyseq_covid19:latest SILVA_test
```

I added config file to root directory that made it use pre built conda environments. but it struggled at step3a. i copied name of env folder because i thought it might be the step3a with wrong name
i copied env-7390a52046feb71a7f6f3361a867269b under name env-84e06c5335c0a958ed012db619fdfceb

I think it worked but i can't get it to run. it crashes but doesn't report ir. so now i want to test it manually:
```bash
docker run -it --rm \
  -v /media/aygera/external_disk/biostar/NCB/attempt2:/workflow \
  jonovox/easyseq_covid19:latest \
  bash
cd /workflow/work/96/b8d54a01a9c21890e58ca517adb45e
conda activate /workflow/conda/env-84e06c5335c0a958ed012db619fdfceb
kma -t_db /workflow/db/SILVA/KMA/SILVA \
    -ipe MetaGV9-A3_R1_fastp.fastq.gz MetaGV9-A3_R2_fastp.fastq.gz \
    -t 8 -ef -ex_mode -1t1 -mq 120 -and -apm f \
    -o MetaGV9-A3_debug
```

Seemes to be working fine. i will change config file to simialrly show any messages by the software used
This is my nextflow.config file:
```code
// Conda environment settings
conda {
    cacheDir = "$baseDir/conda"
}

cleanup = true

process {
    echo = true              // print commands being run
    errorStrategy = 'terminate'  // stop if a task fails
    executor = 'local'       // run jobs on the local machine
}
```

It also struggled to get env for step6a so i copied env-3abca7a24ea4d6c708bf4c6cea6413d2
 under name env-e79cb31e94a201cefc3ec441dba85424/

To run it faster I will use more CPUs:
bash run_batch_docker.sh /home/aygera/biostar/NCB/attempt2/input/ _001.fastq.gz SILVA 24 jonovox/easyseq_covid19:latest SILVA_test

Let's wait a little. Maybe it just needs to finish
In nf script i modified this line
    -ef -ex_mode -1t1 -mq 120 -and -apm f -o ${samplename} 2>/dev/null || exit 0
To become:
    -ef -ex_mode -1t1 -mq 120 -and -apm f -o ${samplename}



Still no progress. thinking about running
```bash
kma -t_db /workflow/db/SILVA/KMA/SILVA -ipe MetaGV9-A3_R1_fastp.fastq.gz MetaGV9-A3_R2_fastp.fastq.gz -t 24 -ef -ex_mode -1t1 -mq 120 -and -apm f -o MetaGV9-A3 
```

manually

Okay so when i run this manually it still gets stuck on finding k mer ankers. idk how much time its going to take. let's wait 40 mins and see. if its still taking a while i will set it with nextflow to run until tomorrow morning but for now ill see if it can at least run one sample till the end of the day. one thing certain it definitely starts running so there is no issue with dependencies
I removed || exit 0 part (from manual testing) because it causes program to report success even if it fails. Nonetheless its still stuck
