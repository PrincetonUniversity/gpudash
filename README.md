# gpudash

The `gpudash` command displays a GPU utilization dashboard for the last hour:

```
usage: gpudash [-h] [-u NETID] [-n] [-c]

GPU utilization dashboard for the last hour

optional arguments:
  -h, --help       show this help message and exit
  -u NETID         create dashboard for a single user
  -n, --no-legend  flag to hide the legend
  -c, --cryoem     flag to show cryoem nodes

Utilization is the percentage of time during a sampling window (< 1 second) that
a kernel was running on the GPU. The format of each entry in the dashboard is
username:utilization (e.g., aturing:90). Utilization varies between 0 and 100%.

Examples:

  Show dashboard for all users:
    $ gpudash

  Show dashboard for the user aturing:
    $ gpudash -u aturing

  Show dashboard for all users without displaying legend:
    $ gpudash -n

  Show dashboard for the cryoem nodes:
    $ gpudash -c

  Show dashboard for the cryoem nodes for user aturing:
    $ gpudash -c -u aturing
```

## Getting Started

The `gpudash` command buiilds on the [https://github.com/PrincetonUniversity/jobstats](Jobstats) platform.

### 1. Create a script to pull data from Prometheus:

```bash
#!/bin/bash

DATA="/home/jdh4/bin/gpus/data"
printf -v SECS '%(%s)T' -1

curl -s 'http://vigilant2.sn17:8480/api/v1/query?query=nvidia_gpu_duty_cycle' > ${DATA}/util.${SECS}
curl -s 'http://vigilant2.sn17:8480/api/v1/query?query=nvidia_gpu_jobUid'     > ${DATA}/uid.${SECS}
curl -s 'http://vigilant2.sn17:8480/api/v1/query?query=nvidia_gpu_jobId'      > ${DATA}/jobid.${SECS}

find ${DATA} -type f -mmin +70 -exec rm -f {} \;

/usr/licensed/anaconda3/2022.5/bin/python /home/jdh4/bin/gpus/extract.py
```

### 2. Create an entry in crontab

### 3. Extract the data from the Prometheus files

With these steps in place, you can use the `gpudash` command:

```
$ gpudash
```

## Getting Help

Please post an issue on this repo. Extensions to the code are welcome.
