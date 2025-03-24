# Artifact description
This is the artifact of submission (**Selective Concolic Testing**) in OOPSLA 2025. 

The artifact is provided as a docker image, which contains the prototype of the proposed method and benchmarks used for evaluation. The aim is to assist the reviewers in reproducing the experimental results in our evaluation.

# Prerequisites
+ Hardware: ~2.50 GHz CPU (all experiments were performed on a server with Intel(R) Xeon(R) Platinum 8269CY CPU @ 2.50GHz, 104 Cores, 192GB), 50+GB free disk space.
+ Unix / Linux OS: We have validated the artifact on Ubuntu system
+ [Docker](https://www.docker.com/pricing/)

# Running the artifact
We assume that the following commands are run in sudo mode. 

Firstly, pull the already prebuilt docker image from [docker hub](https://hub.docker.com/r/oopsla25sct/oopsla25sct). Please make sure its name is `oopsla25sct/oopsla25sct`.
```sh
$ docker pull oopsla25sct/oopsla25sct:v1
```

If everything is ok, a `oopsla25sct/oopsla25sct` image should be found in the images listed by `docker images` command. Then, you can create a container of such image and start a Bash session using the following command. An interactive `bash` shell on the container is also executed at the moment.
```sh
$ docker run -it oopsla25sct/oopsla25sct:v1 bash
```

If all goes well, the container should be running successfully. Otherwise, you can seek help from [Docker Doc](https://docs.docker.com/) if needed. 

Now, navigate into the directory containing our experimental environment and list the contents. 
```sh
$ cd /home/aaa/expriment/ && ls
bench-cephes/  bench-fdlibm/  bench-gsl/  run_expr.py  run_single.py
```
The necessary items to reproduce our evaluation are listed, including benchmark programs and scripts. The rest of this README assumes you are working in `/home/aaa/expriment`.

# Reproducing experimental results

There are 3 benchmarks in our evaluation: GSL (`bench-gsl`) and Cephes (`bench-cephes`) that contains complex non-linear floating-point conditions, and FDLIBM (`bench-fdlibm`) where most programs only contain linear conditions. Hence, we provide a script to reproduce experimental results on these 3 benchmarks. 

In view of the long running time of our evaluation, the scripts try to perform all experiments in parallel. Specifically, the scripts take a user-defined value *n* and exploit at most *n* processes of working machine at run time. The value should be carefully considered to avoid resource exhaustion. In our evaluation, *n* is 80, since all experiments were performed on a 104-core server with 192GB memory.

**Reproduce experimental results:**

```bash
# python3 run_expr.py <parallel num> <execute time>
$ nohup python3 run_expr.py 80 1800 &  
# wait......
```

All tasks will be repeated 5 times, requiring approximately **7200 core-hours**. If you run one CPU core for one hour, that is one core-hour. 

After the analysis is completed, the script will create corresponding files for each benchmark to output the running results, as shown below.

```bash
$ ls
bench-cephes     cephes_fin_covtrend.txt  fdlibm_fin_covtrend.txt  gsl_fin_covtrend.txt  run_expr.py      
bench-fdlibm     cephes_fin.csv           fdlibm_fin.csv           gsl_fin.csv           run_single.py
bench-gsl        cephes_fin_result.txt    fdlibm_fin_result.txt    gsl_fin_result.txt    
```

Each benchmark corresponds to 3 result files (`*_fin.csv`, `*_fin_reslut.txt`, `*_fin_covtrend.txt`). 

Among these 3 files, `*_fin.csv` is a further display of the results of `*_fin_reslut.txt` and also the final coverage table for each benchmark.

The output format of CSV file is as follows:

```
Task name   ,cov-value_predict_1800_0,..., cov-value_smt_1800_0,..., cov-value_jfs_1800_0,... , cov-value_optimize_1800_0,... , random-covnew_smt_1800_0,
-           , line, branch, time,     ..., line, branch, time,  ..., line, branch, time,  ... , line, branch, time,       ... , line, branch, time
gsl_hypot   , 17  , 12    , 16  ,     ..., 17  , 12    , 15  ,  ..., 17  , 12    , 86  ,  ... , 17  , 12    , 15  ,       ... , 17  , 12    , 17
gsl_sf_log_e, 5   , 2     , 13  ,     ..., 5   , 2     , 51  ,  ..., 5   , 2     , 224 ,  ... , 5   , 2     , 48  ,       ... , 5   , 2     , 120
gsl_atanh   , 11  , 7     , 14  ,     ..., 11  , 7     , 300 ,  ..., 11  , 7     , 666 ,  ... , 11  , 7     , 311 ,       ... , 10  , 7     , 84
...... many lines ......
```

And `*_fin_covtrend.txt` shows the number of code lines and branches, which are covered by each method in generating test cases over time under each test case.

The coverage trend txt file is as follows:

```
*******
Benchmark cov trend :/home/aaa/expriment/bench-gsl : cov-value_predict_1800_0
=== TestCase : gsl_sf_hyperg_1F1_e
time:1, line:14, branch:9
time:1, line:14, branch:10
time:2, line:14, branch:11
......
time:1836, line:345, branch:179
=== End
=== TestCase : gsl_cdf_tdist_Pinv
time:1, line:4, branch:2
time:1, line:5, branch:3
......
time:119, line:25, branch:11
=== End
*******
```

Please note that **cov-value_predict** corresponds to the **S** method proposed in this paper, **cov-value_smt** corresponds to the **PS** method, **cov-value_jfs** corresponds to the **PR** method, **cov-value-optimize** corresponds to the **O** method, and **random covnew_smt** corresponds to the **RCN** method

The CSV files (`*_fin.csv`) give the supported data of **Fig. 5** and **Fig. 6**, the coverage files (`*_fin_covtrend.txt`) give the supported data of **Fig. 7** and **Fig. 8**.

# Analyzing single program
If you are interested in a certain benchmark program, e.g., `bench-gsl/gsl_sf_erf_e.c`, please navigate to the `expriment` directory and invoke `run_single.sh` script as follows.

```bash
# python3 run_single.py <parallel num> <execute time> <program path>
$ python3 run_single.py 4 30 bench-gsl/gsl_sf_erf_e.c

Executing ====  /home/aaa/expriment/bench-gsl/gsl_sf_erf_e_cov-value_optimize_30_0_output
Executing ====  /home/aaa/expriment/bench-gsl/gsl_sf_erf_e_cov-value_jfs_30_0_output
Executing ====  /home/aaa/expriment/bench-gsl/gsl_sf_erf_e_cov-value_smt_30_0_output
Executing ====  /home/aaa/expriment/bench-gsl/gsl_sf_erf_e_cov-value_predict_30_0_output
Executing ====  /home/aaa/expriment/bench-gsl/gsl_sf_erf_e_random-covnew_smt_30_0_output
```
The `run_single.sh` script will invoke all of the 5 methods to analyze program `gsl_sf_erf_e.c` under `bench-gsl` folder for 30 seconds, with a parallelism of 4. 

The analyzing results of this program can be found in the working directory, e.g., `bench-gsl/gsl_sf_erf_e_output` and `bench-gsl/gsl_sf_erf_e.runlog`. And the coverage information will be automatically output to the console.

```bash
...... many lines ......

GCov file path : /home/aaa/gsl/specfunc/.libs/erfc.c
====  Replay Ktest ====
Replay KTest : /home/aaa/expriment/bench-gsl/gsl_sf_erf_e_cov-value_jfs_30_0_output/test000001.ktest
  time:0, line:13, branch:3
Replay KTest : /home/aaa/expriment/bench-gsl/gsl_sf_erf_e_cov-value_jfs_30_0_output/test000002.ktest
  time:2, line:48, branch:9
Final ===  48 9 
===================
    Running =====> /home/aaa/expriment/bench-gsl/gsl_sf_erf_e.c
GCov file path : /home/aaa/gsl/specfunc/.libs/erfc.c
====  Replay Ktest ====
Replay KTest : /home/aaa/expriment/bench-gsl/gsl_sf_erf_e_cov-value_optimize_30_0_output/test000001.ktest
  time:0, line:13, branch:3
Replay KTest : /home/aaa/expriment/bench-gsl/gsl_sf_erf_e_cov-value_optimize_30_0_output/test000002.ktest
  time:0, line:48, branch:9
  
 ...... many lines ......
```

The results of a single task will also be output to the corresponding benchmark files (`gsl_fin_covtrend.txt`, `gsl_fin.csv`, `gsl_fin_result.txt`).
