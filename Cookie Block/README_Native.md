
# Artifact Guide for Native Installation - Automating Cookie Consent and GDPR Violation Detection

## Overview
  * [Introduction](##Introduction)
  * [Installation](##Installation)
    * [Requirements](###Requirements)
  * [Reproducing the Dataset Collection](##Reproducing%20the%20Dataset%20Collection)
    * [Crawler Requirements](###Crawler%20Requirements)
    * [Running the CMP Presence Crawler](###Running%20the%20CMP%20Presence%20Crawler)
    * [Running the Consent Data Crawler](###Running%20the%20Consent%20Data%20Crawler)
    * [Extracting the Training Data](###Extracting%20the%20Training%20Data)
    * [Closing Remarks](###Closing%20Remarks)
  * [Feature Extraction and Classifier Results](##Feature%20Extraction%20and%20Classifier%20Results)
    * [Classifier Requirements](###Classifier%20Requirements)
    * [Feature Extraction](###Feature%20Extraction)
    * [XGBoost Classifier](###XGBoost%20Classifier)
    * [Cookiepedia Baseline](###Cookiepedia%20Baseline)
    * [Additional Remarks](###Additional%20Remarks)
  * [Applying the GDPR Violation Detection Scripts](##Applying%20the%20GDPR%20Violation%20Detection%20Scripts)
    * [Additional Statistics](###Additional%20Statistics)
  * [The CookieBlock Extension](##The%20CookieBlock%20Extension)
    * [Recreating the Model Files](###Recreating%20the%20Model%20Files)
    * [Running CookieBlock](###Running%20CookieBlock)
  * [Final Remarks](##Final%20Remarks)
----

## Introduction

This document will guide the reader through the steps required to reproduce the results presented in the paper
"Automating Cookie Consent and GDPR Violation Detection".
This will only consider native installation of the tools on a Linux machine.
If you want to use the provided VM instead, please refer to the other README document.
Note that with the exception of the Browser Extension component, Windows and MacOSX are not officially supported.

Our work consists of four components, each of which can be evaluated separately:
  1. **The Cookie Consent Webcrawlers**
  2. **The Feature Extraction and XGBoost Classifier**
  3. **The GDPR Violation Detection Scripts**
  4. **The Browser Extension: CookieBlock**

This guide will proceed through the components in that order.
We provide prebaked sample inputs in case a step fails, takes too long to execute,
or if it helps to reproduce the results shown in the paper.

The source code of each component is public and MIT licensed, with the exception of
the OpenWPM crawler, which is released under the GPL3 license.

Each component is hosted on its own Github repository:

* Web Crawlers: [https://github.com/dibollinger/CookieBlock-Consent-Crawler](https://github.com/dibollinger/CookieBlock-Consent-Crawler/tree/d8c3991572bf559061c3a12431088d66fc5576e0)
* Classifier:[https://github.com/dibollinger/CookieBlock-Consent-Classifier](https://github.com/dibollinger/CookieBlock-Consent-Classifier/tree/20bdb9a56d34f8859809a9726cb011578d450834)
* Violation Detection: [https://github.com/dibollinger/CookieBlock-Violation-Detection](https://github.com/dibollinger/CookieBlock-Violation-Detection/tree/044c183bc52e8aaf3febe9caa3591be7a13bccac)
* CookieBlock: [https://github.com/dibollinger/CookieBlock](https://github.com/dibollinger/CookieBlock/tree/d610277a370440d717d3942aa89d9c776591cf27)

If the documentation in this README does not suffice, we provide additional documentation
and code comments in the repositories linked above. We also provide a dataset of cookie statistics
as well as prebaked inputs, which are included as part of this package.

Additionally, we include a Virtual Machine image to make reproducing the results as easy as possible.
The Virtual Machine comes with all dependencies pre-installed, and requires no further setup.
Please refer to the other README for VM-specific instructions.

Should you experience any problems with the provided tools, you can report the issues on the respective Github pages.

----
## Installation

The first step is to clone the codebases for each component.

We recommend creating a new folder to store the components in, and then cloning the public Git repositories using the following commands:

**Cookie Consent Webcrawlers:**

```
git clone --branch aec https://github.com/dibollinger/CookieBlock-Consent-Crawler
```

**Feature Extraction and XGBoost Classifier:**

```
git clone --branch aec https://github.com/dibollinger/CookieBlock-Consent-Classifier
```

**GDPR Violation Detection Scripts:**

```
git clone --branch aec https://github.com/dibollinger/CookieBlock-Violation-Detection
```

**CookieBlock Extension:**

```
git clone --branch aec https://github.com/dibollinger/CookieBlock
```

In addition, to reproduce the results, you should also download the dataset that is included as part of this package.

### Requirements

In order to run the components on your Linux machine, you will need to install a Python interpreter,
as well as the `npm` tool for the Javascript components.
Each Python codebase contains a `requirements.txt` file, which specifies the library dependencies for each project.

These dependencies can be installed with the command:
```
python -m pip install -r requirements.txt
```

The Consent Crawler uses OpenWPM, and additionally requires the Conda tool, which can be downloaded from the following URL:

https://docs.conda.io/en/latest/miniconda.html

After installing conda, you will need to install the environment by running:

```
cd CookieBlock-Consent-Crawler/crawler/openwpm
bash install.sh
```

and then activate the environment using

```
conda activate openwpm
```

We recommend installing the conda environment before downloading the python requirements,
so that everything can be cleanly removed afterwards.

Note that the code has only been tested on Linux, specifically on Debian-based distributions.
The OpenWPM version we use does not work on Windows or MacOSX, so to reproduce the
consent data crawl results, a Linux machine is required.

The browser extension itself is standalone, was tested on Linux and Windows, and can be
directly loaded in Chrome by pointing it to the `src` folder of the repository, or in Firefox ESR/Nightly
by loading a zip file of the folder contents as an addon.

The extension is also available for Chrome and Firefox on the official addon stores here:

**Firefox:** https://addons.mozilla.org/en-US/firefox/addon/cookieblock/

**Chrome:** https://chrome.google.com/webstore/detail/cookieblock/fbhiolckidkciamgcobkokpelckgnnol

----

## Reproducing the Dataset Collection

**Corresponding Repository:**
[https://github.com/dibollinger/CookieBlock-Consent-Crawler](https://github.com/dibollinger/CookieBlock-Consent-Crawler/tree/d8c3991572bf559061c3a12431088d66fc5576e0)

In this section we guide you on the use of the dataset collection scripts.
This corresponds to Chapter 2 of the paper, "Dataset Collection".

The results for this step are heavily dependent on the current state of the web. Because websites might
have changed which CMPs they use, or because the CMPs may have altered how they present their data, the
webcrawlers may not work exactly as they did during our evaluation from May 2021.

Moreover, to make sure that CMPs are found on as many domains as
possible, run the crawl from within an EU country, for example, using a VPN in Germany.
Different locations will yield different results.

Executing the following scripts could potentially use a lot of memory and CPU time. The scripts will be sending requests
to external domains that have not been analyzed in great detail. Known malicious domains should however be excluded,
thanks to filtering done by the Tranco list generation. While the process of crawling websites shouldn't cause any issues
on your host, it does send out a lot of network requests, so make sure that you are allowed to perform a webcrawl from
within your network.

### Crawler Requirements

For this step, you will need to install the `conda` cross-platform Python package management tool.
This is a requirement of OpenWPM. Furthermore, the OpenWPM crawler only works on Linux.

Conda can be downloaded from the following URL:

https://docs.conda.io/en/latest/miniconda.html

After installing conda, you will need to install the environment by running:

```
cd CookieBlock-Consent-Crawler/crawler/openwpm
bash install.sh
```

and then activate the environment using

```
conda activate openwpm
```
If this does not work immediately, try restarting your shell.

To run OpenWPM in headless mode, you will also need the `xvfb` package. On Ubuntu, this can be installed with:
```
sudo apt install xvfb
```

The Python dependencies for the remaining scripts can be installed by installing
the libraries listed in `requirements.txt`, with the command:

```
python -m pip install -r requirements.txt
```

Note that you will need to install the python requirements after installing the conda environment.

### Running the CMP Presence Crawler

Given a set of candidate domains, the goal of this crawler is to quickly filter out URLs that
do not use any of the CMPs we target. This includes the CMPs "`Cookiebot`", "`OneTrust`"
(which encompasses the CMPs "`OneTrust`", "`CookiePro`" and "`Optanon`", as they use the same structure),
as well as "`Termly`". This process corresponds to what is described in Section 2.2 of the paper.

1.  Open the repository folder: `CookieBlock-Consent-Crawler/crawler/`

This subfolder contains all scripts that relate directly to crawling the web for cookie data.

For the evaluation shown in the paper, we used a set of almost 6 million input domains. Running this process
took multiple days, therefore we provide a smaller sample input to demonstrate the functionality of
the crawler. These domains were sourced from the website "BuiltWith", and have a much greater probability
of containing the desired CMPs, hence we will receive more results in less time.

2. The sample input file can be found in the folder: `CookieBlock-Consent-Crawler/crawler/samples/unfiltered_domains.txt`

Use this file as input to begin the crawling process.

3. To start the CMP presence crawl with the sample domains as targets, execute the command:
```bash
python run_presence_crawl.py --numthreads 32 --file samples/unfiltered_domains.txt
```
If everything worked correctly, you should now see the following text being printed:
```
Start fast crawl
0/6939
```
The process may take between 10-15 minutes to complete, but this varies depending on machine and website state.
Eventually it may start printing the sites that timed out. This is a normal occurrence as some sites may be
unreachable at times. If the process takes too long, try increasing the number of threads used.

Once the task completes, it will create the following files in the current working directory:
```
filtered_domains/
filtered_domains/bot_responses.txt
filtered_domains/cookiebot_responses.txt
filtered_domains/crawler_timeouts.txt
filtered_domains/failed_urls.txt
filtered_domains/http_responses.txt
filtered_domains/nocmp_responses.txt
filtered_domains/onetrust_responses.txt
filtered_domains/termly_responses.txt
presence_crawl.log
```

`presence_crawl.log` contains the stdout output, and is mainly used for debugging purposes.

The `filtered_domains` folder contains the results of the crawl, split into categories.
* `bot_responses.txt` contains all domains that detected the crawler as being a bot, and prevented access to the site.
* `cookiebot_responses.txt` contains all Cookiebot CMP domains.
* `crawler_timeouts.txt` contains all sites for which the crawler timed out, i.e. no response was received.
* `failed_urls.txt` contains all domains for which DNS resolution failed, or where other network errors occurred.
* `http_responses.txt` contains all domains that responded with an HTTP error (unrelated to bot detection).
* `nocmp_responses.txt` contains all domains that do not use one of the supported CMPs.
* `onetrust_responses.txt` contains all OneTrust CMP domains.
* `termly_responses.txt` contains all Termly CMP domains.

In the resulting dataset, there should be approximately 200(686) OneTrust responses, 260(669) Cookiebot responses, and 50(127) Termly responses.

### Running the Consent Data Crawler

Now that you have a filtered set of domains, you can proceed to use OpenWPM to extract the cookie categories from the CMPs.

4. As input, you can use either the files `cookiebot_responses.txt`, `onetrust_responses.txt` and `termly_responses.txt
   from the previous step, or use the provided sample file `samples/filtered_domains.txt`.
5. Make sure that you have the `(openwpm)` environment active, and then run the command:
```bash
python run_consent_crawl.py all --num_browsers 4 --file filtered_domains/cookiebot_responses.txt --file filtered_domains/onetrust_responses.txt --file filtered_domains/termly_responses.txt
```
or, to use the prebaked set:
```bash
python run_consent_crawl.py all --num_browsers 4 --file samples/filtered_domains.txt
```

This will start 4 Firefox browsers running in headless mode as background processes.
The browsers will visit the specified domains in parallel, and each will collect the cookies found on the website,
and retrieve the cookie declarations from the CMPs while doing so.

If you notice that the command is printing error messages, this may be because `xvfb` is not installed on your system.
Installing the package on Linux will usually resolve the issue:
```
sudo apt install xvfb
```
In the VM, this requirement is already satisfied.

With the approximately 2000 sample domains in the prebaked set, the crawling process can take more than a full day to complete.
The database may also take up a significant amount of space as more data is collected.
Therefore, we recommend interrupting the process after approximately 100 domains were
crawled successfully, using a simple keyboard interrupt.

Collecting 100 domains will take at most 1 hour on the VM.
The output will still be generated and entered into the resulting database, even if the process is aborted early.

Running this script will create the following files:
```
./collected_data/crawl_data_<timestamp>.sqlite
./logs/crawl_<timestamp>.log
```
Where `<timestamp>` stands for a time and date string at which the process finished.
This string is used to prevent overwriting the same file on repeat executions.

The crawl log contains the full debug output detailing the OpenWPM crawl.
The SQLite database contains the collected cookie declarations, as well as the observed cookies.
We recommend the `sqlitebrowser` application to view and explore its contents.
However, this is not strictly necessary as we have scripts that will aggregate and extract the most important content.

On Ubuntu, the tool can be installed by executing the command:
```
sudo apt install sqlitebrowser
sqlitebrowser ./collected_data/crawl_data_<timestamp>.sqlite
```

A detailed documentation on the tables contained within the database can be found in the README file of the repository.

### Extracting the Training Data

6. Browse to the folder `CookieBlock-Consent-Crawler/database_processing/`

For this part of the guide, we will again use a sample file.
In this case, we will use the prepared cookie database, which is found in
```
CookieBlock-Consent-Crawler/database_processing/example_db/example_crawl_20210213_153228.sqlite
```
and contains the results of crawling approximately 300 target websites.

7. Run the command:
```
python post_process_db.py ./example_db/example_crawl_20210213_153228.sqlite
```
This will add some Views to the database, constructed from several queries which will allow you to browse
more useful information using `sqlitebrowser`. Additionally, it will compute several statistics on the
cookie content, such as the total number of cookie records, cookie updates, and the number of cookie labels by class.

It will also output debug information on the error state of the individual website visits of the crawl.
We used this to resolve problems in the design of the crawler.

The statistics are output to:
```
./stats/debug_stats_example_crawl_20210213_153228.txt
./stats/content_stats_example_crawl_20210213_153228.txt
```

IMPORTANT: If instead of the prepared database, you use the database collected in previous steps, and then receive the following error:
```
Traceback (most recent call last):
  File "post_process_db.py", line 378, in <module>
    exit(main())
  File "post_process_db.py", line 370, in main
    extract_content_statistics(conn, content_statistics_path)
  File "post_process_db.py", line 256, in extract_content_statistics
    avg_num_diffs =3D int(cur.fetchone()["avg_diffs"])
TypeError: int() argument must be a string, a bytes-like object or a number, not 'NoneType'
```
Then it is likely that the crawler failed to collect any cookie data in the crawl. If this occurs, verify using `sqlitebrowser` that the database tables are filled.
If not, you may need to rerun the crawl, or switch to the prepared dataset.

8. Second, execute the command:
```
python extract_cookies_from_db.py example_db/example_crawl_20210213_153228.sqlite
```
This will perform the extraction of the training data cookies by matching the declarations `name`,
`domain` and `visit_id` with those of the observed cookies. Since the declarations may list multiple
domains and have minor differences in structure, Python string processing is required to match the records.

The training cookies for the classifier are output to:
```
./training_data_output/example_crawl_20210213_153228.json
```
The script will also log the number of training data entries that were extracted, separated by category.

### Closing Remarks

Feel free to also run the commands listed above with the datasets found in `~/CookieCompliance/Datasets/`.
This will however require more resources, and a lot more time to complete -- which is not necessary to
verify the functionality of the scripts.

* `./Datasets/01_Tranco_Domains/` contains the full 6M domains we sourced from the Tranco list.
  This is the input list we provided to the presence crawler.
* `./Datasets/02_Presence_Crawl_Results/` contains the results of the presence crawl applied to the above domains.
* `./Datasets/03_CMP_Crawl_Input/` contains the input domains we used for the CMP crawl with OpenWPM.
* `./Datasets/04_Cookie_Databases/` contains the resulting databases from 3 separate crawls.
  These are the databases we used as input to the [Violation Detection](##Applying%20the%20GDPR%20Violation%20Detection%20Scripts),
  as well as for the Feature Extraction, which we will explore in the next section.

----
## Feature Extraction and Classifier Results

**Corresponding Repository:**
[https://github.com/dibollinger/CookieBlock-Consent-Classifier](https://github.com/dibollinger/CookieBlock-Consent-Classifier/tree/20bdb9a56d34f8859809a9726cb011578d450834)

To train the XGBoost classifier, the cookies must first be transformed into a vector representation.
Therefore, we transform the training cookies from JSON format into a sparse matrix format, which is done through the
feature extraction. Afterwards, the sparse matrix is provided to the XGBoost classifier, which will train for at most
20 iterations, and output validation statistics as well as the computed model.

This part of the guide corresponds to Sections 3 and 4 of the paper.

### Classifier Requirements

For this section, only Python libraries are required.
Simply install the dependencies listed in `requirements.txt`, which is placed in the base folder.

As input we will use the Training JSON file, which is contained in the prebaked sample inputs that are packaged with this README:
```
/Datasets/06_Training_Data/tranco_05May_20210510_201615.json
```
This is the full dataset of cookies we used for the classifier evaluation.
Unlike the crawler, the feature extraction and the classifier are more efficient, and provide more consistent, reproducible results.

### Feature Extraction

1. Clone the repository we linked above, and change directory to the path `CookieBlock-Consent-Classifier/`.
   Additionally, download the prebaked sample dataset, extract it, and copy the file
   `06_Training_Data/tranco_05May_20210510_201615.json` into the base folder of the repository.

2. Run the following command to begin extracting features from the training data cookies:

```
python prepare_training_data.py tranco_05May_20210510_201615.json
```
This will start the feature extraction process, which will take approximately 10-15 minutes.

The feature extraction proceeds in 3 phases, extracting per-cookie features, per-update features,
and per-difference features, as described in Appendix B of the paper. The complete list of features
that are extracted, as well as corresponding documentation, can be found in the file:
```
CookieBlock-Consent-Classifier/feature_extraction/features.json
```
Note that this file controls which features are extracted from cookies.
You can change the settings of this file to play around with the output of the feature extraction.

After executing the Python script, you should see the following output:
```
Number of cookies loaded: 304422
Output as sparse feature matrix.
Number of per-cookie functions: 21
Number of per-cookie functions enabled: 21
Number of per-update functions: 28
Number of per-update functions enabled: 28
Number of per-diff functions: 3
Number of per-diff functions enabled: 3
Number of Per-Cookie Features: 1572
Number of Per-Update Features: 114
Number of Per-Diff Features: 3
Number of Features Total: 1689
Begin feature transformation...
Begin feature extraction process...
```
As well as a progress bar. These values can help you identify issues in the feature setup.
After the process completes, the script will output debug timings for each feature extraction step.

By default, the script will produce a sparse matrix, which will be output in binary format to the
folder `processed_features/` along with several additional files:
```
processed_features/feature_map_<timestamp>.txt
processed_features/processed_<timestamp>.sparse
processed_features/processed_<timestamp>.sparse.feature_names
processed_features/processed_<timestamp>.sparse.labels
processed_features/processed_<timestamp>.sparse.weights
```

* The feature map file `processed_features/feature_map_<timestamp>.txt` records which features are
  placed at which indices in the feature array, and can be used with XGBoost to output information
  on the most important features (optional, not shown in the paper).
* `processed_<timestamp>.sparse` contains the training data vectors.
* `processed_<timestamp>.sparse.labels` contains the ground truth labels, in the same order as the training data vectors.
* `processed_<timestamp>.sparse.weights` contains input sample weights.
  We counteract the imbalance in the dataset during classifier training using these weights.
* `processed_<timestamp>.sparse.feature_names` contains human-readable feature names for each index.

Note that the four files ending in `.sparse*` must be placed in the same directory,
and must keep the same name, otherwise the classifier scripts will not be able to train the model.

### XGBoost Classifier

To train the XGBoost model, you will need the output of the feature extraction step.

3. Browse to the folder `./CookieBlock-Consent-Classifier/classifiers/` and execute the command:
```
python train_xgb.py ../processed_features/processed_<timestamp>.sparse split
```
Note that you should replace `<timestamp>` with the timestamp of the sparse matrix that was generated in the previous step.

If the command is successful, you should then see the output:
```
classifier :: INFO :: Loading sparse data...
classifier :: INFO :: Loading complete.
classifier.xgboost :: INFO :: Training using train-test split.
```
This will run a single round of an `80/20` train/test split, training the model on 80% of the data, and validating it on the remainder.
The default settings of XGBoost are hereby the ones shown in the paper in the Appendix B, Table 6.

The process will run for approximately 10-15 minutes on the VM. Once completed, it will output a large range
of validation scores to standard output, and to the log file, stored at `train_xgb.log`.

The script will output various validation metrics. The ones that were used in the paper include the following log outputs:
```
classifier :: INFO :: Total Accuracy Count: 48261
classifier :: INFO :: Total Accuracy Ratio: 0.871861
classifier :: INFO :: Micro Precision: 0.871861
classifier :: INFO :: Micro Recall: 0.871861
classifier :: INFO :: Micro F1Score: 0.871861
classifier :: INFO :: Macro Precision: 0.809422
classifier :: INFO :: Macro Recall: 0.845872
classifier :: INFO :: Macro F1Score: 0.821299
classifier :: INFO :: Weighted Precision: 0.884028
classifier :: INFO :: Weighted Recall: 0.871861
classifier :: INFO :: Weighted F1Score: 0.875934
classifier :: INFO :: Precision for each class: [0.87647991 0.52602939 0.89976756 0.93541327]
classifier :: INFO :: Recall for each class: [0.81268908 0.77498664 0.89702142 0.89879195]
classifier :: INFO :: F1Score for each class: [0.84337998 0.62668828 0.89839239 0.91673702]
```

Since the variance is low, even the outputs for a single fold should be consistent, and comparable to
the results we present in the paper. Note that in case an array is listed, this is always supposed to
be interpreted as the cookie classes `[necessary, functional, analytics, advertising]`, in that order.

It also outputs the confusion matrix after each run:
```
2021-10-13-13:49:38 :: classifier :: INFO :: Predicted labels & accuracy when using ARGMAX as a prediction rule
2021-10-13-13:49:38 :: classifier :: INFO :: Confusion Matrix:
[[ 9402  1109   628   430]
 [  416  2900   220   206]
 [  515   556 15871   751]
 [  394   948   920 20088]]
```
The rows hereby represent the true labels, while the columns represent the predictions.
As with the arrays, the order of classes is `[necessary, functional, analytics, advertising]`

The XGBoost model file itself will be written to the directory:
```
./classifiers/models/xgbmodel_<timestamp>.xgb
```

While the validation data will be output to the folder:
```
./classifiers/xgb_predict_stats/validation_matrix_<timestamp>.sparse
```
This can be used to recompute the validation score using the computed model file.

4. To run training without splitting the data simply execute the following command:

```
python train_xgb.py ../processed_features/processed_<timestamp>.sparse train
```

5. To run cross-validation, instead run the following command:
```
python train_xgb.py ../processed_features/processed_<timestamp>.sparse cross_validate
```
This will split the training dataset into 5 folds, producing one validation output for each.
You can compare these outputs to the ones we present in Section 4 of the paper, as well as the
pre-baked cross-validation logs found inside the dataset, in the folder `07_Classifier_Evaluation/xgboost_may17_21/`.

### Cookiepedia Baseline

6. The scripts that were used to retrieve the baseline categories from Cookiepedia can be found in the folder
`CookieBlock-Consent-Crawler/cookie_statistics_analysis/`.

The Cookiepedia labels were queried using the command:
```
python cookie_stats.py 6 ../../Datasets/06_Training_Data/tranco_05May_20210510_201615.json --bypath
```
This script is a crawler that will look up all cookie names found in the given training data file on Cookiepedia,
and return the corresponding category. It produces a file called
`cookiepedia_lookup.pkl` in the same folder as the script, which can in a second step be used to produce the baseline scores.

However, this command may take multiple days to complete.
The reason for this is that the Cookiepedia repository is usually very slow, which often results in timeouts when trying to query the categories.
This is caused by the hosting of Cookiepedia itself, and is not a factor that is under our control.

For the next step, the repository already contains a file called `cookiepedia_lookup_June4_complete.pkl`
which stores a number of cookie names with corresponding Cookiepedia categories, collected starting on June 4th 2021.
This file contains all cookie names present in the prebaked sample inputs.

7. In order to compute the Cookiepedia baseline evaluation, you will need to execute the following command:
```
python cookiepedia_baseline.py ../../Datasets/06_Training_Data/tranco_05May_20210510_201615.json
```
This will run a 5-fold train-test split, where the training step is skipped, and only the validation is applied,
with the Cookiepedia categories from the file `cookiepedia_lookup_June4_complete.pkl` being used as predictors.

The command will output the evaluation to standard output, as well as into a log file.
To verify that the script produced correct results, the same output can be found in the prebaked dataset, with path:
`07_Classifier_Evaluation/cookiepedia_baseline_may17_21/baseline_stats.log`

### Additional Remarks

The classifier folder contains a large number of additional scripts. Some, such as `train_rnn.py` or `train_catboost.py`
were used to try different classifier approaches, which are mentioned in the extended report from the master thesis,
but not in the paper. Said report can be found at:

https://doi.org/10.3929/ethz-b-000477333

Others, such as `xgboost_small_dump.py` serve to transform the model into a different format, usable by the extension.

The reviewer is encouraged to look through the source code and explore the scripts that are placed in the repository.
Most should be reasonably well documented to allow the reader to understand what the purpose of each script is.

In the dataset folder, you will find the directory `07_Classifier_Evaluation`, which contains pre-computed evaluations
on the XGBoost classifier, as well as the Cookiepedia baseline.

----
## Applying the GDPR Violation Detection Scripts

**Corresponding Repository:**
[https://github.com/dibollinger/CookieBlock-Violation-Detection](https://github.com/dibollinger/CookieBlock-Violation-Detection/tree/044c183bc52e8aaf3febe9caa3591be7a13bccac)

Using the GDPR violation detection scripts is straightforward. To reproduce the results shown in section 6 of the paper,
you will need the three databases from the dataset folder `04_Cookie_Databases`:

* `tranco_05May_20210510_201615.sqlite`
* `no_consentomatic_tranco05May_20210512_164029.sqlite`
* `consentomatic-reject-tranco05May-20210514_081933.sqlite`

Clone the repository and open the directory `CookieBlock-Violation-Detection/`.

Then, execute the following commands in sequence:
```
python method1_wrong_label.py ../Datasets/04_Cookie_Databases/tranco_05May_20210510_201615.sqlite

python method2_majority_deviation.py ../Datasets/04_Cookie_Databases/tranco_05May_20210510_201615.sqlite

python method3_inconsistent_expiry.py ../Datasets/04_Cookie_Databases/tranco_05May_20210510_201615.sqlite

python method4_unclassified_cookies.py ../Datasets/04_Cookie_Databases/tranco_05May_20210510_201615.sqlite

python method5_undeclared_cookies.py ../Datasets/04_Cookie_Databases/tranco_05May_20210510_201615.sqlite

python method6_contradictory_labels.py ../Datasets/04_Cookie_Databases/tranco_05May_20210510_201615.sqlite

python method7_implicit_consent.py ../Datasets/04_Cookie_Databases/no_consentomatic_tranco05May_20210512_164029.sqlite

python method8_ignored_choices.py ../Datasets/04_Cookie_Databases/consentomatic-reject-tranco05May-20210514_081933.sqlite
```
* Method 1 to 6 require the database `tranco_05May_20210510_201615.sqlite`,
  and correspond to the novel types of violation we describe in our paper.
* Method 7 requires the database `no_consentomatic_tranco05May_20210512_164029.sqlite`,
  which is essentially the same crawl, but without the Consent-O-Matic extension being active,
  meaning that CMPs are never given consent.
* Method 8 requires the database `no_consentomatic_tranco05May_20210512_164029.sqlite`,
  where Consent-O-Matic was set up to reject all consent option, rather than accept.

These commands correspond to the violation detection methods listed in Sections 6.1 to 6.4.

Each command should not take more than a few minutes to execute on the VM. Each command will output a JSON and
a txt document, containing the cookie data, and the domains respectively, that potentially violate the GDPR.

Each command will also output the number of cookies that are potential violations, the number of sites with potential
violations, and several other statistics.

Note that arrays such as the example below always correspond to the following categories:
```
[85709, 18779, 88279, 111498, 10584, 689, 5819]
[necessary, functional, analytics, advertising, uncategorized, social media, unknown]
```
The social media category has been omitted for the paper.
Furthermore, CMP types are always listed in the order of `[Cookiebot, OneTrust, Termly]`.

The resulting documents are written to the folder `./violation_stats/`,
while the console output is logged in the file `detector.log`.

### Additional Statistics

Finally, after running all the above commands, move to the folder
`./violation_stats/` and execute the command:
```
python violation_stats.py
```
This will parse the results of the previous commands and output a large number of additional statistics.

Some of these have been used for the paper, such as the histogram in Figure 6:
```
Exactly 7 potential violations: 10 -- 0.034%
Exactly 6 potential violations: 345 -- 1.174%
Exactly 5 potential violations: 1550 -- 5.272%
Exactly 4 potential violations: 5320 -- 18.096%
Exactly 3 potential violations: 7336 -- 24.954%
Exactly 2 potential violations: 8166 -- 27.777%
Exactly 1 potential violations: 5123 -- 17.426%
```

We provide pre-baked files with these outputs as well, which are found in the folder `05_GDPR_Violations`.
You can compare the results to the ones in the paper and in the dataset.

----
## The CookieBlock Extension

**Corresponding Repository:**
[https://github.com/dibollinger/CookieBlock](https://github.com/dibollinger/CookieBlock/tree/d610277a370440d717d3942aa89d9c776591cf27)

The CookieBlock extension repository is split into two parts:
* The node feature extraction
* The extension source code

The node feature extraction is required due to technical reasons. For one, it is not possible for us to use
the XGBoost model directly in the extension code. For the other, we need to use the same feature extraction code
that is used online as part of the browser extension to also extract features for the training cookies.
This is done to prevent minor differences in the feature extraction between Python and JavaScript from affecting
the results of the model predictions.

The following provides a guide on how the forest of decision trees used by the model can be replicated and integrated
into CookieBlock. Finally, we will briefly touch upon how the extension can be installed and used.

This part corresponds to Section 5 of the paper.

### Recreating the Model Files

In this guide, we will be reconstructing the minified XGBoost model files that are used as part of the extension.
These models are stored in the form of `json` files, rather than binary `xgb` files.
This is done for one to allow the extension to parse the decision trees, as well as to reduce the overall filesize
of the models, and the extension.

Precomputed model files can already be found inside the folder:
```
CookieBlock/src/ext_data/model
```

1. Clone the repository linked above. The node feature extraction is stored in the folder:
```
CookieBlock/nodejs-feature-extractor
```

2. Clone the classifier repository (if you haven't done so already), which is needed to train the model.

3. Download the dataset (if you haven't done so already) and copy the file `06_Training_Data/tranco_05May_20210510_201615.json`
   into the base folder of the feature extractor.

4. To run the node feature extraction, you will first need to run the following command in
   the extractor folder to install the necessary dependencies:
```
npm install
```
5. If the previous step was successful, executing the following command from within the folder `CookieBlock/nodejs-feature-extractor`
   will start the extraction process:
```
./cli.js extract tranco_05May_20210510_201615.json
```

6. If everything worked correctly, you should now be seeing the output:
```
Feature extraction mode selected.
Reading data from input json document at: ../../Datasets/06_Training_Data/tranco_05May_20210510_201615.json
Input parsing completed.
Clearing old outputs.
Writing feature map...
Feature map written to path: ./outputs/feature_map.txt
Writing class weight file...
Class weights written to path: ./outputs/class_weights.txt
Performing feature extraction...
```
The process may take 10-15 minutes to finish. Once completed, the resulting sparse matrix will be stored
in the form of a `LibSVM` file at the path:`./outputs/processed.libsvm`, as well as the weights file `./outputs/class_weights.txt`

7. Next, copy the output from the previous step to the classifier repository:
```
CookieBlock-Consent-Classifier/classifiers
```
and execute the command
```
python train_xgb.py processed.libsvm train class_weights.txt
```
to train an XGB model on the JavaScript-extracted features. This produces another model in `./models/`.

8. Finally, run the command
```
python xgboost_small_dump.py models/xgbmodel_<timestamp>.xgb
```
using the model you created in the previous step, which will create a minimal JSON dump of the forest of
decision trees stored inside the XGBoost model file.

The files are stored in the following paths of the classifier folder:
```
./minimal_dump/forest_class0.json
./minimal_dump/forest_class1.json
./minimal_dump/forest_class2.json
./minimal_dump/forest_class3.json
```
which is one forest for each of the four categories of cookies, `necessary`, `functional`, `analytics`, and `advertising`, in that order.

These models can optionally be copied to the path:
```
CookieBlock/src/ext_data/model
```
These models form the core of CookieBlock. Without it, the extension could not function.

Note that copying these models is not necessary, as the repository already comes with precomputed decision tree forests.

### Running CookieBlock

In order to install the extension for Firefox ESR/Nightly, simply pack the files inside the folder:
```
CookieBlock/src
```
into a zip archive, and then load it inside Firefox' extension settings.
Note that the mainline Firefox builds only accept signed extensions, therefore, either Firefox ESR
or Firefoy Nightly is necessary to run the extension.

For Chromium-based browsers, one only needs to enable extension debug mode, and then point the browser to the folder
```
CookieBlock/src
```
to install the extension.
A signed version of the CookieBlock extension is also available on the Firefox, Chrome, Edge and Opera addon stores.

On the top right of the browser bar, you will notice CookieBlock's logo in the extension listing.
Clicking on it will bring up a popup, where one can whitelist the current site or view additional settings.
The settings pages are thoroughly documented, and we will not elaborate on them here.

In order to view additional details about what the extension does in the background, open the `chrome://extensions` page,
click the `Details` button, and inspect the view `background/cookieblock_background.html`. Here, you can now open the console
and view debug messages about what the extension is doing (note that you may need to change the verbosity of the console output to "debug").

For example, we can see what cookies the extension has classified:
```
Perform Prediction: Cookie (CONSENT;.google.com;/) receives label (Necessary)
Perform Prediction: Cookie (_ga;.facebook.com;/) receives label (Analytical)
```

Finally, in the dataset folder `08_CookieBlock_Evaluation` we included the evaluation
of CookieBlock on 100 randomly sampled websites. Because this evaluation was done manually,
we do not have a script to reproduce these results, but we instead provide the dataset by itself.

----
## Final Remarks

There are additional scripts placed in the repositories that this guide did not touch on.
Most of these are not relevant for the paper, or are self-explanatory in nature.

For completeness we also included the files used to create the figures and graphs found in the paper,
and placed them inside the dataset folder `~/CookieConsent/Datasets/09_PDF_Graphics/`.

We hope that this guide was useful in helping you find your way around the different artifacts
that allowed us to build the CookieBlock extension, and which helped in creating the paper
"Automating Cookie Consent and GDPR Violation Detection".

---

Authors of this document:
* Dino Bollinger (ETH Zürich)
* Karel Kubicek (ETH Zürich)

Authors of the paper:
* Dino Bollinger (ETH Zürich)
* Karel Kubicek (ETH Zürich)
* Carlos Cotrini (ETH Zürich)
* David Basin (ETH Zürich)
