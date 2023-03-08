# gpudash

The `gpudash` command is part of the [Jobstats platform](https://github.com/PrincetonUniversity/jobstats). The `gpudash` command displays a GPU utilization dashboard in text (no graphics) for the last hour:

![gpudash example](images/gpudash.png)

The dashboard can be generated for a specific user:

![gpudash user example](images/gpudash_user.png)

Here is the help menu:

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

The `gpudash` command buiilds on the [Jobstats platform](https://github.com/PrincetonUniversity/jobstats). To run the software it requires Python 3.6+ and version 1.17+ of the Python `blessed` package.

### 1. Create a script to pull data from Prometheus

The `query_prometheus.sh` Bash script below makes three queries to Prometheus. Old files are removed. The `extract.py` Python script is called to extract the data and write columns files. The column files are read by `gpudash`.

```bash
$ cat query_prometheus.sh
#!/bin/bash

printf -v SECS '%(%s)T' -1
DATA='/path/to/gpudash/data'
PROM_QUERY='http://vigilant2.sn17:8480/api/v1/query?'

curl -s ${PROM_QUERY}query=nvidia_gpu_duty_cycle > ${DATA}/util.${SECS}
curl -s ${PROM_QUERY}query=nvidia_gpu_jobUid     > ${DATA}/uid.${SECS}
curl -s ${PROM_QUERY}query=nvidia_gpu_jobId      > ${DATA}/jobid.${SECS}

# remove any files that are greater or equal to 70 minutes old
find ${DATA} -type f -mmin +70 -exec rm -f {} \;

# extract the data from the Prometheus queries, convert UIDs to usernames, write column files
python3 /path/to/extract.py
```

The above script will generate column files with the format:

```
$ head -n 5 column.1
{"timestamp": "1678144802", "host": "comp-g1", "index": "0", "user": "ft130", "util": "92", "jobid": "46034275"}
{"timestamp": "1678144802", "host": "comp-g1", "index": "1", "user": "ft130", "util": "99", "jobid": "46015684"}
{"timestamp": "1678144802", "host": "comp-g1", "index": "2", "user": "ft130", "util": "99", "jobid": "46015684"}
{"timestamp": "1678144802", "host": "comp-g1", "index": "3", "user": "kt415", "util": "44", "jobid": "46048505"}
{"timestamp": "1678144802", "host": "comp-g2", "index": "0", "user": "kt415", "util": "82", "jobid": "46015407"}
```

The column files are read by `gpudash` to generate the dashboard.

### 2. Generate a CSV file called `uid2user.csv` containing UIDs and the corresponding usernames. Here is a sample of the file:

```
$ head -n 5 uid2user.csv
153441,ft130
150919,lc455
224256,sh235
329819,bb274
347117,kt415
```

The above file can be generated by running the following command:

```bash
$ getent passwd | awk -F":" '{print $3","$1}' > /path/to/uid2user.txt
```

### 3. Create two entries in crontab

```bash
0,10,20,30,40,50 * * * * /path/to/query_prometheus.sh > /dev/null 2>&1
0 6 * * 1 getent passwd | awk -F":" '{print $3","$1}' > /path/to/uid2user.csv 2> /dev/null
```

The first entry above calls the script that queries the Prometheus server every 10 minutes. The second entry creates the CSV files of UIDs and usernames.

### 4. Download gpudash

`gpudash` is a pure Python code. It's only dependency is the `blessed` Python package. On Ubuntu Linux, this can be installed with:

```bash
$ apt-get install python-blessed
```

Then put `gpudash` in a local like `/usr/local/bin`:

```
$ cd /usr/local/bin
$ wget https://raw.githubusercontent.com/PrincetonUniversity/gpudash/main/gpudash
$ chmod 755 gpudash
```

Next, edit `gpudash` by replacing `cluster1` with the beginning of the cluster name. Modify `all_nodes` to generate a list of compute node names. Lastly, set `SBASE` to the path containing the column files produced by `extract.py`.

With these steps in place, you can use the `gpudash` command:

```
$ gpudash
```

## Troubleshooting

The two most commons problems are (1) setting the correct paths throughout the procedure and (2) installing the Python `blessed` package. You also need to specify the nodes to be displayed by `gpudash`.


## Getting Help

Please post an issue on this repo. Extensions to the code are welcome.
