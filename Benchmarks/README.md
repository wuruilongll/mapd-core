# MapD Benchmark Script

Python scripts that leverages [pymapd](https://github.com/omnisci/pymapd) to query a mapd database and reports performance of queries or data import.

## Setup

Scripts are intended for use on Python3

All required python packages can be installed with `pip` using the `requirements.txt` file:
```
pip install -r requirements.txt
```

The required packages can otherwise be installed individually using [conda](https://conda.io/) or [pip](https://pypi.org/project/pip/). List of required packages:

1) [pymapd](https://github.com/omnisci/pymapd) - Provides a python DB API 2.0-compliant OmniSci interface (formerly MapD)

   Install with conda: `conda install -c conda-forge pymapd` or with pip: `pip install pymapd`

2) [pandas](https://pandas.pydata.org/) - Provides high-performance, easy-to-use data structures and data analysis tools

   Install with conda: `conda install pandas` or with pip: `pip install pandas`

3) [numpy](http://www.numpy.org/) - Package for scientific computing with Python

   Install with conda: `conda install -c anaconda numpy` or with pip: `pip install numpy`

4) [nvidia-ml-py3](https://github.com/nicolargo/nvidia-ml-py3) - Python bindings to the NVIDIA Management Library

   Install with conda: `conda install -c fastai nvidia-ml-py3` or with pip: `pip install nvidia-ml-py3`

## Usage

### Required components

Running the script requies a few components:

1) Connection to a mapd db with a dataset loaded - either a large sample dataset used for benchmarking, or a custom dataset.
2) Query(ies) to run against the dataset. These can be provided as files with the following syntax: `query_<query_id>.sql` (where query_id is a unique query ID). The script will find all queries from files that match the syntax in the directory passed in to the script at runtime.
3) Destination. Depending on the type of destination, a connection to a mapd db, or destination file location may be required.

#### Running the script

Usage can be printed at any time by running: `./run-benchmark.py -h` or `--help`

Currently, usage is:

```
usage: run-benchmark.py [-h] [-v] [-q] [-u USER] [-p PASSWD] [-s SERVER]
                        [-o PORT] [-n NAME] -t TABLE -l LABEL [-d QUERIES_DIR]
                        -i ITERATIONS [-g GPU_COUNT] [-G GPU_NAME]
                        [--no-gather-conn-gpu-info]
                        [--no-gather-nvml-gpu-info] [--gather-nvml-gpu-info]
                        [-m MACHINE_NAME] [-a MACHINE_UNAME] [-e DESTINATION]
                        [-U DEST_USER] [-P DEST_PASSWD] [-S DEST_SERVER]
                        [-O DEST_PORT] [-N DEST_NAME] [-T DEST_TABLE]
                        [-C DEST_TABLE_SCHEMA_FILE] [-j OUTPUT_FILE_JSON]
                        [-J OUTPUT_FILE_JENKINS] [-E OUTPUT_TAG_JENKINS]

required arguments:
  -u USER, --user USER  Source database user
  -p PASSWD, --passwd PASSWD
                        Source database password
  -s SERVER, --server SERVER
                        Source database server hostname
  -n NAME, --name NAME  Source database name
  -t TABLE, --table TABLE
                        Source db table name
  -l LABEL, --label LABEL
                        Benchmark run label
  -d QUERIES_DIR, --queries-dir QUERIES_DIR
                        Absolute path to dir with query files. [Default:
                        "queries" dir in same location as script]
  -i ITERATIONS, --iterations ITERATIONS
                        Number of iterations per query. Must be > 1

optional arguments:
  -h, --help            show this help message and exit
  -v, --verbose         Turn on debug logging
  -q, --quiet           Suppress script outuput (except warnings and errors)
  -o PORT, --port PORT  Source database server port
  -g GPU_COUNT, --gpu-count GPU_COUNT
                        Number of GPUs. Not required when gathering local gpu
                        info
  -G GPU_NAME, --gpu-name GPU_NAME
                        Name of GPU(s). Not required when gathering local gpu
                        info
  --no-gather-conn-gpu-info
                        Do not gather source database GPU info fields
                        [run_gpu_count, run_gpu_mem_mb] using pymapd
                        connection info. Use when testing a CPU-only server.
  --no-gather-nvml-gpu-info
                        Do not gather source database GPU info fields
                        [gpu_driver_ver, run_gpu_name] from local GPU using
                        pynvml. Defaults to True when source server is not
                        "localhost". Use when testing a CPU-only server.
  --gather-nvml-gpu-info
                        Gather source database GPU info fields
                        [gpu_driver_ver, run_gpu_name] from local GPU using
                        pynvml. Defaults to True when source server is
                        "localhost". Only use when benchmarking against same
                        machine that this script is run from.
  -m MACHINE_NAME, --machine-name MACHINE_NAME
                        Name of source machine
  -a MACHINE_UNAME, --machine-uname MACHINE_UNAME
                        Uname info from source machine
  -e DESTINATION, --destination DESTINATION
                        Destination type: [mapd_db, file_json, output,
                        jenkins_bench] Multiple values can be input seperated
                        by commas, ex: "mapd_db,file_json"
  -U DEST_USER, --dest-user DEST_USER
                        Destination mapd_db database user
  -P DEST_PASSWD, --dest-passwd DEST_PASSWD
                        Destination mapd_db database password
  -S DEST_SERVER, --dest-server DEST_SERVER
                        Destination mapd_db database server hostname (required
                        if destination = "mapd_db")
  -O DEST_PORT, --dest-port DEST_PORT
                        Destination mapd_db database server port
  -N DEST_NAME, --dest-name DEST_NAME
                        Destination mapd_db database name
  -T DEST_TABLE, --dest-table DEST_TABLE
                        Destination mapd_db table name
  -C DEST_TABLE_SCHEMA_FILE, --dest-table-schema-file DEST_TABLE_SCHEMA_FILE
                        Destination table schema file. This must be an
                        executable CREATE TABLE statement that matches the
                        output of this script. It is required when creating
                        the results table. Default location is in
                        "./results_table_schemas/query-results.sql"
  -j OUTPUT_FILE_JSON, --output-file-json OUTPUT_FILE_JSON
                        Absolute path of .json output file (required if
                        destination = "file_json")
  -J OUTPUT_FILE_JENKINS, --output-file-jenkins OUTPUT_FILE_JENKINS
                        Absolute path of jenkins benchmark .json output file
                        (required if destination = "jenkins_bench")
  -E OUTPUT_TAG_JENKINS, --output-tag-jenkins OUTPUT_TAG_JENKINS
                        Jenkins benchmark result tag. Optional, appended to
                        table name in "group" field
```

Example 1:
```
python ./run-benchmark.py -t flights_2008_10k -l TestLabel -d /data/queries/flights -i 10 -g 4 -S localhost
```
this would run the script with the following parameters:
- Default values for source mapd db: localhost, default username and password, and database name
- Queries would be run against the "flights_2008_10k" table
- Results would have label "TestLabel"
- Query file(s) would be sources from directory "/data/queries/flights"
- Query(ies) would run for 10 iterations each
- Results would show that mapd_db machine has 4 GPUs
- Destination mapd db is located at localhost with the default username, password, and database name.

Example 2:
```
python ./run-benchmark.py -u user -p password -s mapd-server.example.com -n mapd_db -t flights_2008_10k -l TestLabel -d /home/mapd/queries/flights -i 10 -g 4 -e mapd_db,file_json -U dest_user -P password -S mapd-dest-server.mapd.com -N mapd_dest_db -T benchmark_results -j /home/mapd/benchmark_results/example.json
```

## Import Benchmark

The import benchmark script `./run-benchmark-import.py` is used to run a data import from a file local to the benchmarking machine, and report various times associated with the import of that data.

### Usage

#### Required components

Running the script requies a few components:

1) Connection to a db, can be local or remote.
2) Data source file(s) on the machine with the database.
3) Schemas associates with each data source file to create the table before performing the import. These must be executable .sql files kept in the `./import_table_schemas` directory.
3) Destination for the results. Depending on the type of destination, a connection to a mapd db, or destination file location may be required.

#### Running the script

Usage can be printed at any time by running: `./run-benchmark-import.py -h` or `--help`

Currently, usage is:

```
usage: run-benchmark-import.py [-h] [-v] [-q] [-u USER] [-p PASSWD]
                               [-s SERVER] [-o PORT] [-n NAME] -l LABEL -f
                               IMPORT_FILE -c TABLE_SCHEMA_FILE
                               [-t IMPORT_TABLE_NAME]
                               [-F IMPORT_QUERY_TEMPLATE_FILE]
                               [--no-drop-table-before]
                               [--no-drop-table-after] [-A IMPORT_TEST_NAME]
                               [-m MACHINE_NAME] [-a MACHINE_UNAME]
                               [-e DESTINATION] [-U DEST_USER]
                               [-P DEST_PASSWD] [-S DEST_SERVER]
                               [-O DEST_PORT] [-N DEST_NAME] [-T DEST_TABLE]
                               [-C DEST_TABLE_SCHEMA_FILE]
                               [-j OUTPUT_FILE_JSON] [-J OUTPUT_FILE_JENKINS]

required arguments:
  -u USER, --user USER  Source database user
  -p PASSWD, --passwd PASSWD
                        Source database password
  -s SERVER, --server SERVER
                        Source database server hostname
  -n NAME, --name NAME  Source database name
  -l LABEL, --label LABEL
                        Benchmark run label
  -f IMPORT_FILE, --import-file IMPORT_FILE
                        Absolute path to file on omnisci_server machine with
                        data for import test
  -c TABLE_SCHEMA_FILE, --table-schema-file TABLE_SCHEMA_FILE
                        Path to local file with CREATE TABLE sql statement for
                        the import table

optional arguments:
  -h, --help            show this help message and exit
  -v, --verbose         Turn on debug logging
  -q, --quiet           Suppress script outuput (except warnings and errors)
  -o PORT, --port PORT  Source database server port
  -t IMPORT_TABLE_NAME, --import-table-name IMPORT_TABLE_NAME
                        Name of table to import data to. NOTE: This table will
                        be dropped before and after the import test, unless
                        --no-drop-table-[before/after] is specified.
  -F IMPORT_QUERY_TEMPLATE_FILE, --import-query-template-file IMPORT_QUERY_TEMPLATE_FILE
                        Path to file containing template for import query. The
                        script will replace "##TAB##" with the value of
                        import_table_name and "##FILE##" with the value of
                        table_schema_file. By default, the script will use the
                        COPY FROM command with the default default delimiter
                        (,).
  --no-drop-table-before
                        Do not drop the import table and recreate it before
                        import NOTE: Make sure existing table schema matches
                        import .csv file schema
  --no-drop-table-after
                        Do not drop the import table after import
  -A IMPORT_TEST_NAME, --import-test-name IMPORT_TEST_NAME
                        Name of import test (ex: "ips"). Required when using
                        jenkins_bench_json as output.
  -m MACHINE_NAME, --machine-name MACHINE_NAME
                        Name of source machine
  -a MACHINE_UNAME, --machine-uname MACHINE_UNAME
                        Uname info from source machine
  -e DESTINATION, --destination DESTINATION
                        Destination type: [mapd_db, file_json, output,
                        jenkins_bench] Multiple values can be input seperated
                        by commas, ex: "mapd_db,file_json"
  -U DEST_USER, --dest-user DEST_USER
                        Destination mapd_db database user
  -P DEST_PASSWD, --dest-passwd DEST_PASSWD
                        Destination mapd_db database password
  -S DEST_SERVER, --dest-server DEST_SERVER
                        Destination mapd_db database server hostname (required
                        if destination = "mapd_db")
  -O DEST_PORT, --dest-port DEST_PORT
                        Destination mapd_db database server port
  -N DEST_NAME, --dest-name DEST_NAME
                        Destination mapd_db database name
  -T DEST_TABLE, --dest-table DEST_TABLE
                        Destination mapd_db table name
  -C DEST_TABLE_SCHEMA_FILE, --dest-table-schema-file DEST_TABLE_SCHEMA_FILE
                        Destination table schema file. This must be an
                        executable CREATE TABLE statement that matches the
                        output of this script. It is required when creating
                        the results table. Default location is in
                        "./results_table_schemas/query-results.sql"
  -j OUTPUT_FILE_JSON, --output-file-json OUTPUT_FILE_JSON
                        Absolute path of .json output file (required if
                        destination = "file_json")
  -J OUTPUT_FILE_JENKINS, --output-file-jenkins OUTPUT_FILE_JENKINS
                        Absolute path of jenkins benchmark .json output file
                        (required if destination = "jenkins_bench")
```

### Additional details

1) Import query template file: If the import command needs to be customized - for example, to use a delimiter other than comma - an import query template file can be used. This file must contain an executable query with two variables that will be replaced by the script: a) ##TAB## will be replaced with the import table name, and b) ##FILE## will be replaced with the import data file.
