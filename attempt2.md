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
```nf
    -ef -ex_mode -1t1 -mq 120 -and -apm f -o ${samplename} 2>/dev/null || exit 0
```
To become:
```nf
    -ef -ex_mode -1t1 -mq 120 -and -apm f -o ${samplename}
```


Still no progress. thinking about running
```bash
kma -t_db /workflow/db/SILVA/KMA/SILVA -ipe MetaGV9-A3_R1_fastp.fastq.gz MetaGV9-A3_R2_fastp.fastq.gz -t 24 -ef -ex_mode -1t1 -mq 120 -and -apm f -o MetaGV9-A3 
```

manually

Okay so when i run this manually it still gets stuck on finding k mer ankers. idk how much time its going to take. let's wait 40 mins and see. if its still taking a while i will set it with nextflow to run until tomorrow morning but for now ill see if it can at least run one sample till the end of the day. one thing certain it definitely starts running so there is no issue with dependencies
I removed || exit 0 part (from manual testing) because it causes program to report success even if it fails. Nonetheless its still stuck

It's been 40 mins. still not finished, not even on 1 sample. I will pause it nw to finish some other work and then run it overnight with nf script

After about 16 hours it finished on all samples. however, the next step didn't work because it needs kaleido version 0.2.1:
opening container:
docker run -it --rm \
  -v /home/aygera/biostar/NCB/attempt2:/workflow \
  jonovox/easyseq_covid19:latest \
  bash

inside it:
conda activate /workflow/conda/env-a1b65765c2bf2d3f21605d8abc9f0cd9
pip install kaleido==0.2.1
This installed it into python 3.8
so now need to:
/workflow/conda/env-a1b65765c2bf2d3f21605d8abc9f0cd9/bin/pip install --force-reinstall kaleido==0.2.1

That worked. Now environment for jinja2, matplotlib, pandas
I copied env-3abca7a24ea4d6c708bf4c6cea6413d2 (step6) as env-3aeb9b79e220d9768a96c230b95029a2 
cp -r env-3abca7a24ea4d6c708bf4c6cea6413d2/ ./env-3aeb9b79e220d9768a96c230b95029a2

Inside container:
/workflow/conda/env-3aeb9b79e220d9768a96c230b95029a2/bin/pip install pandas

Now it lacks permissions for final_script.py:
chmod +x scripts/final_report.py

Now it needs pandas in /workflow/conda/env-3aeb9b79e220d9768a96c230b95029a2. So it's the same env. why doesn't it see it? Might be because i was uin another enviroment while using pip
/workflow/conda/env-3aeb9b79e220d9768a96c230b95029a2/bin/python -m pip install --force-reinstall pandas
/workflow/conda/env-3aeb9b79e220d9768a96c230b95029a2/bin/python -c "import pandas; print(pandas.__version__, pandas.__file__)"

Now it needs weasyprint. I used that env that i got from i dont know where because it has weasyprint. now want to validate if it has pandas
/workflow/conda/env-025066a104bf8ce5621e328d8009733a/bin/python -c "import pandas; print(pandas.__version__, pandas.__file__)"
/workflow/conda/env-025066a104bf8ce5621e328d8009733a/bin/python -c "import matplotlib; print(matplotlib.__version__, matplotlib.__file__)"
/workflow/conda/env-025066a104bf8ce5621e328d8009733a/bin/python -c "import jinja2; print(jinja2.__version__, jinja2.__file__)"

It has all 3. now need to install weasyprint
/workflow/conda/env-025066a104bf8ce5621e328d8009733a/bin/python -m pip install --force-reinstall weasyprint
/workflow/conda/env-025066a104bf8ce5621e328d8009733a/bin/python -c "import weasyprint; print(weasyprint.__version__, weasyprint.__file__)"

Now copying it to have the necesary name:
cp -r env-025066a104bf8ce5621e328d8009733a/ ./env-3aeb9b79e220d9768a96c230b95029a2

For some reason the error is there. Let's validate if necesary env has all 4:
/workflow/conda/env-3aeb9b79e220d9768a96c230b95029a2/bin/python -c "import pandas; print(pandas.__version__, pandas.__file__)"
/workflow/conda/env-3aeb9b79e220d9768a96c230b95029a2/bin/python -c "import matplotlib; print(matplotlib.__version__, matplotlib.__file__)"
/workflow/conda/env-3aeb9b79e220d9768a96c230b95029a2/bin/python -c "import jinja2; print(jinja2.__version__, jinja2.__file__)"
/workflow/conda/env-3aeb9b79e220d9768a96c230b95029a2/bin/python -c "import weasyprint; print(weasyprint.__version__, weasyprint.__file__)"

Still no weasyprint. Let's install it in here:
/workflow/conda/env-3aeb9b79e220d9768a96c230b95029a2/bin/python -m pip install --force-reinstall weasyprint

It now needs docker-wide installations:
apt-get update && apt-get install -y \
    libglib2.0-0 \
    libcairo2 \
    libpango-1.0-0 \
    libpangocairo-1.0-0 \
    libgdk-pixbuf2.0-0 \
    shared-mime-info
Didn't work. Let's google:
sudo apt-get install build-essential python-dev python-pip libcairo2 libpango1.0-0 libgdk-pixbuf2.0-0 libffi-dev shared-mime-info

Looking in documentation for weasyprint:
WeasyPrint ‘v60.2’ (version i have) depends on:
Python ≥ 3.7.0
Pango ≥ 1.44.0
pydyf ≥ 0.6.0
CFFI ≥ 0.6
html5lib ≥ 1.1
tinycss2 ≥ 1.0.0
cssselect2 ≥ 0.1
Pyphen ≥ 0.9.1
Pillow ≥ 9.1.0
fontTools ≥ 4.0.0

I have Python 3.8.5, pango-view (pango) 1.42.4 
I can install these into conda env:
/workflow/conda/env-3aeb9b79e220d9768a96c230b95029a2/bin/python -m pip install Pillow
Have to check if they exist first and check versions

/workflow/conda/env-3aeb9b79e220d9768a96c230b95029a2/bin/python -c "import pydyf; print(pydyf.__version__, pydyf.__file__)"
/workflow/conda/env-3aeb9b79e220d9768a96c230b95029a2/bin/python -c "import cffi; print(cffi.__version__, cffi.__file__)"
/workflow/conda/env-3aeb9b79e220d9768a96c230b95029a2/bin/python -c "import html5lib; print(html5lib.__version__, html5lib.__file__)"
/workflow/conda/env-3aeb9b79e220d9768a96c230b95029a2/bin/python -c "import tinycss2; print(tinycss2.__version__, tinycss2.__file__)"
/workflow/conda/env-3aeb9b79e220d9768a96c230b95029a2/bin/python -c "import cssselect2; print(cssselect2.__version__, cssselect2.__file__)"
/workflow/conda/env-3aeb9b79e220d9768a96c230b95029a2/bin/python -c "import pyphen; print(pyphen.__version__, pyphen.__file__)"
/workflow/conda/env-3aeb9b79e220d9768a96c230b95029a2/bin/python -c "import PIL; import PIL.Image; print(PIL.__version__)"
/workflow/conda/env-3aeb9b79e220d9768a96c230b95029a2/bin/python -c "import fontTools; print(fontTools.__version__, fontTools.__file__)"
All available

pango-view --version
That's too old. need to upgrade to pango 1.44.0
For pango dependencies are:
GObject—2.0
HarfBuzz—0.0

For GObject-2.0 dependencies are:
GLib—2.0

That needs GTK, which needs meson. so starting first:
sudo apt-get install python3 python3-pip python3-setuptools python3-wheel ninja-build
pip3 install --user meson
export PATH=$PATH:/.local/bin
source ~/.bashrc
meson setup --prefix /opt/gtk builddir
CPPFLAGS="-I/opt/gtk/include"
LDFLAGS="-L/opt/gtk/lib"
PKG_CONFIG_PATH="/opt/gtk/lib/pkgconfig"
export CPPFLAGS LDFLAGS PKG_CONFIG_PATH

But then i read somwhere thatit unnecesary. so i tried
sudo apt-get install libgtk-3-dev

conda activate conda/env-3aeb9b79e220d9768a96c230b95029a2
