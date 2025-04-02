# Vertex AI: workbench

Vertex AI is a GCP platform for machine learning (ML) that lets users to
store ML models, features, and training sets and for running their ML
applications. This can be particularly useful for projects that are
fully developed and can be reconfigured for Vertex AI platform. Vertex
AI also provides a workbench environment for testing and developing new
projects. Here, we discuss how to apply compute engines from Vertex AI
to run our workflows on GCP.

------------------------------------------------------------------------

## Create a workbench from GCP console

- From Vertex AI select Workbench
- Select “USER-MANAGED NOTEBOOKS” and press “+ NEW NOTEBOOK”
- Select Python 3 and click “ADVANCED OPTIONS” and set the following
  options and press “CREATE”:
  - Notebook name: select a name
  - Region: `us-central1 (Iowa)` and Zone: `us-central1-a`
- Machine configuration
  - Machin type: select resources that you need
- Networking:
  - Select the shared network if available (if any)
- Permission:
  - If you create the workbench only for yourself then, Select “Single
    user only”. Note that if you select this option other member of your
    group will not be able to use this workbench
  - Use any service account that is available (if any)

**Important Note:** after the instance is created it will cost your team
as long as it is running, no matter if it is using or not. If you do not
use the instance simply **stop** the instance and **start** it when you
need the resources again.

## Using the GCP instance from Jupyter-Lab

After creating an instance (or start an existing one), click “OPEN
JUPYTERLAB” to start the Jupyter Server on the browser. To open a
Notebook, click Python 3 kernel under the Notebook.

### Install new packages or tools

We can install new Python packages form Notebook by using `pip`:

``` bash
# Install Python packages via pip
!pip install <package-name>
```

For instance:

``` bash
# Install pandas-qbq
!pip install pandas-gbq
```

For installing a list of packages (`requirements.txt`), then we can run:

``` bash
# Install Python packages via pip from req. file
!pip install -r requirements.txt
```

If you need to pass a proxy, you can use:

``` bash
# Install Python packages with a proxy
!pip install --proxy <proxy:port> <package-name> 
```

Beside Python packages, users have `sudo` privilege to install/update
Unix tools. For instance, to update current tools run:

``` bash
# Install Unix tools
sudo apt update && sudo -E apt upgrade -y
```

If need a proxy, we can run:

``` bash
# Install Unix tools
!export http_proxy=<proxt:port> && sudo -E apt update && sudo -E apt upgrade -y
```

For an example let’s install Java and add PySpark package:

``` bash
# Install PySpark
sudoapt install -y default-jre
!pip install pyspark
import pyspark
```

### BigQuery

From the Jupyter Notebook users can apply `google.cloud` package to
communicate with public or private datasets in BigQuery. For instance,
run the following to count rows in `penguins` table from a public data:

``` python
# BigQuery from a public data
from google.cloud import bigquery

client = bigquery.Client()
sql = "SELECT COUNT(*) AS COUNT FROM bigquery-public-data.ml_datasets.penguins"
client.query(sql).to_dataframe()['COUNT'].tolist()[0]
```

Or the following for running a query from a private table in our
project:

``` python
# BigQuery from a data that landed in our project
sql = "SELECT DISTINCT TOWN AS TOWN FROM BQ_Tutorial.boston_data"
client.query(sql).to_dataframe().sort_values(by = 'TOWN')
```

Note that `FROM` clause includes
`<project_name>.<dataset_name>.<table_name>`. However, `project_name`
can be skipped if the dataset is in the current project.

The results can be written in the project BigQuery tables. For instance:

``` python
# Write to BigQuery
import pandas_gbq

sql = """
SELECT * FROM bigquery-public-data.ml_datasets.penguins
WHERE  body_mass_g < 3500 AND sex = 'FEMALE'
"""
df = client.query(sql).to_dataframe()
df.to_gbq('BQ_Tutorial.penguins_data', if_exists = 'append') # Will append if table exists. Other options are 'fail' or 'replace'
```

Also, it is possible to directly read data from our buckets. For
instance:

``` python
# Read data from the project bucket
import pandas as pd

file_path_r = 'gs://bucket_read_test/boston_data.csv'
pd.read_csv(file_path_r, sep = '\t')
```

And we can write data to a bucket, such as:

``` python
# Write data to the project bucket
file_path_w = 'gs://bucket_read_test/penguins_data.csv'
df.to_csv(file_path_w)
```

Note that the bucket path has this pattern:
`gs://<bucket_name>/<file_name>`.

## Use the GPC instance from a local terminal

We can use GCP resources directly from our computer’s terminal. For
communicating with GCP resources from terminal, `gcloud` should be
installed in your computer. On Linux (WSL), `gcloud` can be installed
by:

``` sh
# Install Gcloud
echo "deb [signed-by=/usr/share/keyrings/cloud.google.gpg] https://packages.cloud.google.com/apt cloud-sdk main" | sudo tee -a /etc/apt/sources.list.d/google-cloud-sdk.list
curl https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key --keyring /usr/share/keyrings/cloud.google.gpg add -
sudo apt update
sudo apt install -y google-cloud-cli
```

Now, we can login to the GCP instance and run the workflow. To do so,
first run:

``` sh
gcloud auth login --update-adc 
```

Then set the project id and the compute region (if you did not before):

``` sh
gcloud config set project <project_id>
gcloud config set compute/region us-central1-a
gcloud config set compute/zone us-central1-a
```

And finally create a workflow including the following steps: - Start a
GCP instance - Run the script (e.g., Python) - Store the results (e.g.,
in BigQuery table, a bucket, or local) - Stop the instance

For instance, the following script, called `verai-job.sh`, does the
above steps to run a Python script and store results in local:

``` bash
#!/usr/bin/bash
# =============================================================================
# Author: Ashkan Mirzaee
# License: GPL-3.0
# Date: 2023/01/30
# Source: https://github.com/ashki23/HPC-notes
# =============================================================================

## Instance and bucket names as input
instance_name=$1
script_name=$2

## Start instance
gcloud compute instances start $instance_name

## Transfer the Python script to the GCP instance
gcloud compute scp $script_name $instance_name:~ --internal-ip

## Run the job on GCP
gcloud compute ssh $instance_name --internal-ip --command "export PATH=\$PATH:/opt/conda/bin && source activate base && python $script_name"

## Transfer results to local
gcloud compute scp $instance_name:~/result.csv . --internal-ip

## Stop the instance
gcloud compute instances stop $instance_name
```

Where the Python script, called `python-job.py`, is:

``` python
#!/usr/bin/env python
# =============================================================================
# Author: Ashkan Mirzaee
# License: GPL-3.0
# Date: 2023/01/30
# Source: https://github.com/ashki23/HPC-notes
# =============================================================================

from google.cloud import bigquery

client = bigquery.Client()
sql = """
SELECT * FROM bigquery-public-data.ml_datasets.penguins
WHERE  body_mass_g < 3500 AND sex = 'FEMALE'
"""
df = client.query(sql).to_dataframe()
df.to_csv('./result.csv')
```

Now run the workflow on a Vertex AI workbench by:

``` bash
source verai-job.sh <workbench_name> <script_name>
```

For instance:

``` bash
source verai-job.sh notebook-test python-job.py
```

After the above job is finished you can find the results (here
`result.csv`) in the current directory.

---

Copyright, [Ashkan Mirzaee](https://ashki23.github.io/index.html) | Content is available under [CC BY-SA 3.0](https://creativecommons.org/licenses/by-sa/3.0/) | Sourcecode licensed under [GPL-3.0](https://www.gnu.org/licenses/gpl-3.0.en.html)
