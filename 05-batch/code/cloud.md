## Running Spark in the Cloud

### Connecting to Google Cloud Storage 

Uploading data to GCS:

```bash
gsutil -m cp -r pq/ gs://dtc_data_lake_de-zoomcamp-nytaxi/pq
```

Download the jar for connecting to GCS to any location (e.g. the `lib` folder):

**Note**: For other versions of GCS connector for Hadoop see [Cloud Storage connector ](https://cloud.google.com/dataproc/docs/concepts/connectors/cloud-storage#connector-setup-on-non-dataproc-clusters).

```bash
gsutil cp gs://hadoop-lib/gcs/gcs-connector-hadoop3-2.2.5.jar ./lib/
```

See the notebook with configuration in [09_spark_gcs.ipynb](09_spark_gcs.ipynb)

(Thanks Alvin Do for the instructions!)


### Local Cluster and Spark-Submit

Creating a stand-alone cluster ([docs](https://spark.apache.org/docs/latest/spark-standalone.html)):

```bash
./sbin/start-master.sh
```

Creating a worker:

```bash
URL="spark://de-zoomcamp.europe-west1-b.c.de-zoomcamp-nytaxi.internal:7077"
./sbin/start-slave.sh ${URL}

# for newer versions of spark use that:
#./sbin/start-worker.sh ${URL}
```

Turn the notebook into a script:

```bash
jupyter nbconvert --to=script 06_spark_sql.ipynb
```

Edit the script and then run it:

```bash 
python 06_spark_sql.py \
    --input_green=data/pq/green/2020/*/ \
    --input_yellow=data/pq/yellow/2020/*/ \
    --output=data/report-2020
```

Use `spark-submit` for running the script on the cluster

```bash
URL="spark://de-zoomcamp.europe-west1-b.c.de-zoomcamp-nytaxi.internal:7077"

spark-submit \
    --master="${URL}" \
    06_spark_sql.py \
        --input_green=data/pq/green/2021/*/ \
        --input_yellow=data/pq/yellow/2021/*/ \
        --output=data/report-2021
```

### Data Proc

Upload the script to GCS:

```bash
gsutil -m cp -r 06_spark_sql.py gs://kestra-de-zoomcamp-bucket99/code/spark_sql.py
```

Params for the job:

* `--input_green=gs://kestra-de-zoomcamp-bucket99/pq/pq/green/2021/*/`
* `--input_yellow=gs://kestra-de-zoomcamp-bucket99/pq/pq/yellow/2021/*/`
* `--output=gs://kestra-de-zoomcamp-bucket99/report-2021`


Using Google Cloud SDK for submitting to dataproc
([link](https://cloud.google.com/dataproc/docs/guides/submit-job#dataproc-submit-job-gcloud))

```bash
gcloud dataproc jobs submit pyspark \
    --cluster=dezoomcamp-cluster \
    --region=europe-west2 \
    gs://kestra-de-zoomcamp-bucket99/code/spark_sql.py \
    -- \
        --input_green=gs://kestra-de-zoomcamp-bucket99/pq/pq/green/2020/*/ \
        --input_yellow=gs://kestra-de-zoomcamp-bucket99/pq/pq/yellow/2020/*/ \
        --output=gs://kestra-de-zoomcamp-bucket99/report-2020
```

### Big Query

Upload the script to GCS:

```bash
gsutil -m cp -r spark_sql_big_query.py gs://kestra-de-zoomcamp-bucket99/code/spark_sql_big_query.py
```

Write results to big query ([docs](https://cloud.google.com/dataproc/docs/tutorials/bigquery-connector-spark-example#pyspark)):

```bash
gcloud dataproc jobs submit pyspark \
    --cluster=dezoomcamp-cluster \
    --region=europe-west2 \
    --jars=gs://spark-lib/bigquery/spark-3.3-bigquery-0.42.2.jar \
    gs://kestra-de-zoomcamp-bucket99/code/spark_sql_big_query.py \
    -- \
        --input_green=gs://kestra-de-zoomcamp-bucket99/pq/pq/green/2020/*/ \
        --input_yellow=gs://kestra-de-zoomcamp-bucket99/pq/pq/yellow/2020/*/ \
        --output=de_zoomcamp.reports-2020
```

There can be issue with latest Spark version and the Big query connector. Download links to the jar file for respective Spark versions can be found at:
[Spark and Big query connector](https://github.com/GoogleCloudDataproc/spark-bigquery-connector)

**Note**: Dataproc on GCE 2.1+ images pre-install Spark BigQquery connector: [DataProc Release 2.2](https://cloud.google.com/dataproc/docs/concepts/versioning/dataproc-release-2.2). Therefore, no need to include the jar file in the job submission.
