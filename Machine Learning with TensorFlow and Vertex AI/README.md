# [Machine Learning with TensorFlow and Vertex AI](https://www.skills.google/focuses/3391?parent=catalog)

## GSP273

![Google Cloud self-paced labs logo](https://cdn.qwiklabs.com/GMOHykaqmlTHiqEeQXTySaMXYPHeIvaqa2qHEzw6Occ%3D)

---

## Overview

In this lab you create a Vertex AI Workbench instance on which you develop a TensorFlow model in Jupyter notebook. You train the model, create an input data pipeline, deploy it to an endpoint, and get predictions.

**TensorFlow** is an end-to-end open source platform for machine learning. It has a comprehensive, flexible ecosystem of tools, libraries, and community resources that lets researchers push the state-of-the-art in ML and developers easily build and deploy ML powered applications.

**Vertex AI** brings AutoML and AI Platform together into a unified API, client library, and user interface. With Vertex AI, both AutoML training and custom training are available options.

**Vertex AI Workbench** helps users quickly build end-to-end notebook-based workflows through deep integration with data services (like Dataproc, Dataflow, BigQuery, and Dataplex) and Vertex AI. It enables data scientists to connect to Google Cloud data services, analyze datasets, experiment with different modeling techniques, deploy trained models into production, and manage MLOps through the model lifecycle.

This lab uses a set of code samples and scripts developed for _[Data Science on the Google Cloud Platform, 2nd Edition](https://www.oreilly.com/library/view/data-science-on/9781098118945/)_ from O'Reilly Media, Inc.

---

### Objectives

- Deploy Vertex AI Workbench instance
- Create minimal training and validation data
- Create the input data pipeline
- Create TensorFlow model
- Deploy model to Vertex AI
- Deploy Explainable AI model to Vertex AI
- Make predictions from the model endpoint

---

## Setup and Requirements

### Before You Click the Start Lab Button

Read these instructions. Labs are timed and you cannot pause them. The timer, which starts when you click **Start Lab**, shows how long Google Cloud resources are made available to you.

This hands-on lab lets you do the lab activities in a real cloud environment, not in a simulation or demo environment. It does so by giving you new, temporary credentials you use to sign in and access Google Cloud for the duration of the lab.

**Requirements:**

- Access to a standard internet browser (Chrome browser recommended)
- Time to complete the lab—remember, once you start, you cannot pause a lab

**Notes:**

- Use an Incognito or private browser window to run this lab to prevent conflicts between your personal account and the student account
- Use only the student account for this lab. If you use a different Google Cloud account, you may incur charges to that account

---

### How to Start Your Lab and Sign in to the Google Cloud Console

1. Click the **Start Lab** button. If you need to pay for the lab, a dialog opens for you to select your payment method. On the left is the **Lab Details** pane with the following:

   - The Open Google Cloud console button
   - Time remaining
   - The temporary credentials that you must use for this lab
   - Other information, if needed, to step through this lab

2. Click **Open Google Cloud console** (or right-click and select **Open Link in Incognito Window** if you are running the Chrome browser)

   **Tip:** Arrange the tabs in separate windows, side-by-side.

3. If you see the **Choose an account** dialog, click **Use Another Account**

4. Copy the **Username** from the Lab Details pane and paste it into the **Sign in** dialog, then click **Next**

5. Copy the **Password** from the Lab Details pane and paste it into the **Welcome** dialog, then click **Next**

   **Important:** You must use the credentials the lab provides you. Do not use your Google Cloud account credentials.

6. Click through the subsequent pages:
   - Accept the terms and conditions
   - Do not add recovery options or two-factor authentication (because this is a temporary account)
   - Do not sign up for free trials

After a few moments, the Google Cloud console opens in this tab.

**Note:** To access Google Cloud products and services, click the **Navigation menu** or type the service or product name in the **Search** field.

![Navigation menu icon and Search field](https://cdn.qwiklabs.com/9Fk8NYFp3quE9mF%2FilWF6%2FlXY9OUBi3UWtb2Ne4uXNU%3D)

---

## Task 1. Deploy Vertex AI Workbench Instance

### Enable Notebook API

1. In the Google Cloud console, navigate to **APIs & Services > Library**
2. Search for **Notebook API** and click **Enable** if it is not enabled

### Create Workbench Instance

1. In the Google Cloud Console, on the **Navigation menu**, click **Vertex AI > Workbench**
2. Click **+ Create New**
3. In the **Create instance** dialog, use the default name or enter a unique name for the Vertex AI Workbench Instance. Set the region to `region` and zone to `zone` and leave the rest of the settings as default
4. Click **Create**
5. Click **Open JupyterLab**

**Tip:** To use a Notebook, you enter commands into a cell. Run the commands by either pressing **Shift + Enter**, or clicking the triangle on the Notebook top menu to **Run selected cells and advance**.

✅ **Check my progress:** Deploy Vertex AI Workbench instance

---

## Task 2. Create Minimal Training and Validation Data

### Create Notebook

1. In the **Notebook** launcher section click **TensorFlow 2-11 (Local)** to open a new notebook

   ![The Python 3 tile highlighted in the launcher](https://cdn.qwiklabs.com/pZuaGco2dAnXakXMAne9yFGcP57Z4Fr7ygi82UeS%2FO0%3D)

### Import Libraries and Set Environment Variables

2. Import Python libraries and set environment variables:

```python
import os
import json
import math
import shutil
import numpy as np
import tensorflow as tf

# Install Graphviz
!sudo apt install graphviz -y

# Set environment variables
PROJECT = !(gcloud config get-value project)
PROJECT = PROJECT[0]
REGION = ''
BUCKET = f'{PROJECT}-dsongcp'

os.environ['ENDPOINT_NAME'] = 'flights'
os.environ['BUCKET'] = BUCKET
os.environ['REGION'] = REGION
os.environ['TF_VERSION'] = '2-' + tf.__version__[2:4]
```

**Note:** When pasting commands into the Jupyter notebook cell, remember to run the cell before you proceed to the next step.

You can ignore any TensorRT Library Load or TensorFlow AVX2 FMA warnings.

---

### Export Files That Contain Training and Validation Data

When the lab spins up, a few tables are created in the BigQuery dataset. In this section, you use BigQuery to create temporary tables that contain the data you need, then export the tables to CSV files on Google Cloud Storage. You then delete the temporary tables and process the CSV data files to create the training, validation, and full datasets for a custom TensorFlow model.

#### Create Training Dataset

1. Create training dataset `flights_train_data` for model training:

```sql
%%bigquery
CREATE OR REPLACE TABLE dsongcp.flights_train_data AS

SELECT
  IF(arr_delay < 15, 1.0, 0.0) AS ontime,
  dep_delay,
  taxi_out,
  distance,
  origin,
  dest,
  EXTRACT(hour FROM dep_time) AS dep_hour,
  IF (EXTRACT(dayofweek FROM dep_time) BETWEEN 2 AND 6, 1, 0) AS is_weekday,
  UNIQUE_CARRIER AS carrier,
  dep_airport_lat,
  dep_airport_lon,
  arr_airport_lat,
  arr_airport_lon
FROM dsongcp.flights_tzcorr f
JOIN dsongcp.trainday t
  ON f.FL_DATE = t.FL_DATE
WHERE
  f.CANCELLED = False AND
  f.DIVERTED = False AND
  is_train_day = 'True'
```

#### Create Evaluation Dataset

2. Create the evaluation dataset `flights_eval_data` for model evaluation:

```sql
%%bigquery
CREATE OR REPLACE TABLE dsongcp.flights_eval_data AS

SELECT
  IF(arr_delay < 15, 1.0, 0.0) AS ontime,
  dep_delay,
  taxi_out,
  distance,
  origin,
  dest,
  EXTRACT(hour FROM dep_time) AS dep_hour,
  IF (EXTRACT(dayofweek FROM dep_time) BETWEEN 2 AND 6, 1, 0) AS is_weekday,
  UNIQUE_CARRIER AS carrier,
  dep_airport_lat,
  dep_airport_lon,
  arr_airport_lat,
  arr_airport_lon
FROM dsongcp.flights_tzcorr f
JOIN dsongcp.trainday t
  ON f.FL_DATE = t.FL_DATE
WHERE
  f.CANCELLED = False AND
  f.DIVERTED = False AND
  is_train_day = 'False'
```

#### Create Full Dataset

3. Create the full dataset `flights_all_data`:

```sql
%%bigquery
CREATE OR REPLACE TABLE dsongcp.flights_all_data AS

SELECT
  IF(arr_delay < 15, 1.0, 0.0) AS ontime,
  dep_delay,
  taxi_out,
  distance,
  origin,
  dest,
  EXTRACT(hour FROM dep_time) AS dep_hour,
  IF (EXTRACT(dayofweek FROM dep_time) BETWEEN 2 AND 6, 1, 0) AS is_weekday,
  UNIQUE_CARRIER AS carrier,
  dep_airport_lat,
  dep_airport_lon,
  arr_airport_lat,
  arr_airport_lon,
  IF (is_train_day = 'True',
      IF(ABS(MOD(FARM_FINGERPRINT(CAST(f.FL_DATE AS STRING)), 100)) < 60, 'TRAIN', 'VALIDATE'),
      'TEST') AS data_split
FROM dsongcp.flights_tzcorr f
JOIN dsongcp.trainday t
  ON f.FL_DATE = t.FL_DATE
WHERE
  f.CANCELLED = False AND
  f.DIVERTED = False
```

#### Export Datasets to Cloud Storage

4. Export the training, validation, and full datasets to CSV format on Google Cloud Storage:

   **This will take about 2 minutes to complete.**

5. Run the following bash script in your notebook cell:

```bash
%%bash
PROJECT=$(gcloud config get-value project)
for dataset in "train" "eval" "all"; do
  TABLE=dsongcp.flights_${dataset}_data
  CSV=gs://${BUCKET}/ch9/data/${dataset}.csv
  echo "Exporting ${TABLE} to ${CSV} and deleting table"
  bq --project_id=${PROJECT} extract --destination_format=CSV $TABLE $CSV
  bq --project_id=${PROJECT} rm -f $TABLE
done
```

6. List exported objects to Google Cloud Storage bucket:

```python
!gsutil ls -lh gs://{BUCKET}/ch9/data
```

✅ **Check my progress:** Create training and validation dataset

---

## Task 3. Create the Input Data

### Setup in Notebook

1. For development purposes, train for a few epochs:

```python
DEVELOP_MODE = True
NUM_EXAMPLES = 5000 * 1000
```

2. Assign your training and validation data URI:

```python
training_data_uri = f'gs://{BUCKET}/ch9/data/train*'
validation_data_uri = f'gs://{BUCKET}/ch9/data/eval*'
```

3. Set up model parameters:

```python
NBUCKETS = 5
NEMBEDS = 3
TRAIN_BATCH_SIZE = 64
DNN_HIDDEN_UNITS = '64,32'
```

---

### Reading Data into TensorFlow

1. Read the CSV files from Google Cloud Storage into TensorFlow using the `tf.data` package:

```python
if DEVELOP_MODE:
    train_df = tf.data.experimental.make_csv_dataset(training_data_uri, batch_size=5)
    for n, data in enumerate(train_df):
        numpy_data = {k: v.numpy() for k, v in data.items()}
        print(n, numpy_data)
        if n == 1:
            break
```

The `read_dataset()` function reads the training data, yielding `batch_size` examples each time, and allows you to stop iterating once a certain number of examples have been read.

The dataset contains all the columns in the CSV file, named according to the header line. The data consists of both features and the label. Separate them using the `features_and_labels()` function to make the later code easier to read.

2. Create helper functions for data processing:

```python
def features_and_labels(features):
    label = features.pop('ontime')
    return features, label

def read_dataset(pattern, batch_size, mode=tf.estimator.ModeKeys.TRAIN, truncate=None):
    dataset = tf.data.experimental.make_csv_dataset(pattern, batch_size, num_epochs=1)
    dataset = dataset.map(features_and_labels)
    if mode == tf.estimator.ModeKeys.TRAIN:
        dataset = dataset.shuffle(batch_size * 10)
        dataset = dataset.repeat()
        dataset = dataset.prefetch(1)
    if truncate is not None:
        dataset = dataset.take(truncate)
    return dataset

if DEVELOP_MODE:
    print("Checking input pipeline")
    one_item = read_dataset(training_data_uri, batch_size=2, truncate=1)
    print(list(one_item))  # should print one batch of 2 items
```

---

## Task 4. Create, Train and Evaluate TensorFlow Model

### Create Feature Columns

Typically you create one feature for every column in your tabular data. Keras supports feature columns, opening up the ability to represent structured data using standard feature engineering techniques like embedding, bucketizing, and feature crosses.

1. Create real and sparse feature columns:

```python
import tensorflow as tf

# Real-valued columns
real = {
    colname: tf.feature_column.numeric_column(colname)
    for colname in (
        'dep_delay,taxi_out,distance,dep_hour,is_weekday,'
        'dep_airport_lat,dep_airport_lon,'
        'arr_airport_lat,arr_airport_lon'
    ).split(',')
}

# Sparse (categorical) columns
sparse = {
    'carrier': tf.feature_column.categorical_column_with_vocabulary_list(
        'carrier',
        vocabulary_list='AS,VX,F9,UA,US,WN,HA,EV,MQ,DL,OO,B6,NK,AA'.split(',')
    ),
    'origin': tf.feature_column.categorical_column_with_hash_bucket('origin', hash_bucket_size=1000),
    'dest': tf.feature_column.categorical_column_with_hash_bucket('dest', hash_bucket_size=1000),
}
```

### Create Input Layers

All these features come directly from the input file and are provided by any client that wants a prediction for a flight. Input layers map 1:1 to the input features and their types.

2. Create input layers for each column:

```python
# Input layers for real-valued columns
inputs = {
    colname: tf.keras.layers.Input(name=colname, shape=(), dtype='float32')
    for colname in real.keys()
}

# Input layers for sparse columns
inputs.update({
    colname: tf.keras.layers.Input(name=colname, shape=(), dtype='string')
    for colname in sparse.keys()
})
```

---

### Bucketing and Feature Crosses

Real-valued columns whose precision is overkill (thus, likely to cause overfitting) can be discretized and made into categorical columns. Discretize the latitudes and longitudes and cross the buckets—this results in breaking up the country into grids.

3. Create bucketized and crossed features:

```python
# Define latitude and longitude buckets for USA
latbuckets = np.linspace(20.0, 50.0, NBUCKETS).tolist()
lonbuckets = np.linspace(-120.0, -70.0, NBUCKETS).tolist()

# Bucketize latitude and longitude columns
disc = {}
disc.update({
    f'd_{key}': tf.feature_column.bucketized_column(real[key], latbuckets)
    for key in ['dep_airport_lat', 'arr_airport_lat']
})
disc.update({
    f'd_{key}': tf.feature_column.bucketized_column(real[key], lonbuckets)
    for key in ['dep_airport_lon', 'arr_airport_lon']
})

# Cross columns that make sense in combination
sparse['dep_loc'] = tf.feature_column.crossed_column(
    [disc['d_dep_airport_lat'], disc['d_dep_airport_lon']],
    NBUCKETS * NBUCKETS
)
sparse['arr_loc'] = tf.feature_column.crossed_column(
    [disc['d_arr_airport_lat'], disc['d_arr_airport_lon']],
    NBUCKETS * NBUCKETS
)
sparse['dep_arr'] = tf.feature_column.crossed_column(
    [sparse['dep_loc'], sparse['arr_loc']],
    NBUCKETS ** 4
)

# Embed all the sparse columns
embed = {
    f'embed_{colname}': tf.feature_column.embedding_column(col, NEMBEDS)
    for colname, col in sparse.items()
}
real.update(embed)

# One-hot encode the sparse columns
sparse = {
    colname: tf.feature_column.indicator_column(col)
    for colname, col in sparse.items()
}

if DEVELOP_MODE:
    print(sparse.keys())
    print(real.keys())
```

---

### Train and Evaluate the Model

#### Prepare Model Directory

1. Set the output directory for model artifacts:

```python
output_dir = f'gs://{BUCKET}/ch9/trained_model'
os.environ['OUTDIR'] = output_dir  # needed for deployment
print(f'Writing trained model to {output_dir}')
```

2. Delete any existing model checkpoints:

```bash
!gsutil -m rm -rf $OUTDIR
```

**Note:** This may report an error if no files exist in the target location, which is expected.

#### Build Wide-and-Deep Model

3. Create a `wide_and_deep_classifier` function:

```python
def wide_and_deep_classifier(inputs, linear_feature_columns, dnn_feature_columns, dnn_hidden_units):
    # Deep part
    deep = tf.keras.layers.DenseFeatures(dnn_feature_columns, name='deep_inputs')(inputs)
    layers = [int(x) for x in dnn_hidden_units.split(',')]
    for layerno, numnodes in enumerate(layers):
        deep = tf.keras.layers.Dense(
            numnodes,
            activation='relu',
            name=f'dnn_{layerno+1}'
        )(deep)

    # Wide part
    wide = tf.keras.layers.DenseFeatures(linear_feature_columns, name='wide_inputs')(inputs)

    # Combine wide and deep
    both = tf.keras.layers.concatenate([deep, wide], name='both')
    output = tf.keras.layers.Dense(1, activation='sigmoid', name='pred')(both)

    # Create and compile model
    model = tf.keras.Model(inputs, output)
    model.compile(
        optimizer='adam',
        loss='binary_crossentropy',
        metrics=['accuracy']
    )
    return model

# Build the model
model = wide_and_deep_classifier(
    inputs,
    linear_feature_columns=sparse.values(),
    dnn_feature_columns=real.values(),
    dnn_hidden_units=DNN_HIDDEN_UNITS
)

# Visualize model architecture
tf.keras.utils.plot_model(model, 'flights_model.png', show_shapes=False, rankdir='LR')
```

![Model graph](https://cdn.qwiklabs.com/NmtQgrKHGDUfDQSq%2BjoBE0IFFERw8gj86Hc1iclC5FQ%3D)

#### Train the Model

4. Prepare training and evaluation datasets, then train the model:

```python
# Set training parameters
train_batch_size = TRAIN_BATCH_SIZE
if DEVELOP_MODE:
    eval_batch_size = 100
    steps_per_epoch = 3
    epochs = 2
    num_eval_examples = eval_batch_size * 10
else:
    eval_batch_size = 100
    steps_per_epoch = NUM_EXAMPLES // train_batch_size
    epochs = 10
    num_eval_examples = eval_batch_size * 100

# Create datasets
train_dataset = read_dataset(training_data_uri, train_batch_size)
eval_dataset = read_dataset(
    validation_data_uri,
    eval_batch_size,
    tf.estimator.ModeKeys.EVAL,
    num_eval_examples
)

# Set up checkpoint callback
checkpoint_path = f'{output_dir}/checkpoints/flights.cpt'
shutil.rmtree(checkpoint_path, ignore_errors=True)
cp_callback = tf.keras.callbacks.ModelCheckpoint(
    checkpoint_path,
    save_weights_only=True,
    verbose=1
)

# Train the model
history = model.fit(
    train_dataset,
    validation_data=eval_dataset,
    epochs=epochs,
    steps_per_epoch=steps_per_epoch,
    callbacks=[cp_callback]
)
```

#### Visualize Training Results

5. Visualize the model loss and accuracy:

```python
import matplotlib.pyplot as plt

nrows = 1
ncols = 2
fig = plt.figure(figsize=(10, 5))

for idx, key in enumerate(['loss', 'accuracy']):
    ax = fig.add_subplot(nrows, ncols, idx + 1)
    plt.plot(history.history[key])
    plt.plot(history.history[f'val_{key}'])
    plt.title(f'model {key}')
    plt.ylabel(key)
    plt.xlabel('epoch')
    plt.legend(['train', 'validation'], loc='upper left')
```

![The model loss and model accuracy line charts](https://cdn.qwiklabs.com/IqU%2FiPsXgFCIgm9gxWSPQ8sEGye1%2BAnem6qr%2B1av9kI%3D)

**Note:** Training loss and the model accuracy graph may not match exactly because you are training on a very small random sample.

---

### Export the Trained Model

6. Save the model artifacts to Google Cloud Storage:

```python
import time

export_dir = f'{output_dir}/export/flights_{time.strftime("%Y%m%d-%H%M%S")}'
print(f'Exporting to {export_dir}')
tf.saved_model.save(model, export_dir)
```

✅ **Check my progress:** Create the TensorFlow Model

---

## Task 5. Deploy Flights Model to Vertex AI

Vertex AI provides a fully managed, autoscaling, serverless environment for Machine Learning models. You get the benefits of paying for compute resources (such as CPUs or GPUs) only when you are using them. Because the models are containerized, dependency management is taken care of. The Endpoints take care of traffic splits, allowing you to do A/B testing in a convenient way.

Once you deploy the model to Vertex AI, you get capabilities like explainability, drift detection, and monitoring without any additional code.

### Create and Deploy Model Endpoint

1. Create the model endpoint `flights` and deploy the model:

```bash
%%bash

# TF_VERSION and ENDPOINT_NAME set in first cell
TIMESTAMP=$(date +%Y%m%d-%H%M%S)
MODEL_NAME=${ENDPOINT_NAME}-${TIMESTAMP}
EXPORT_PATH=$(gsutil ls ${OUTDIR}/export | tail -1)
echo $EXPORT_PATH

# Create the model endpoint for deploying the model
if [[ $(gcloud ai endpoints list --region=$REGION \
        --format='value(DISPLAY_NAME)' --filter=display_name=${ENDPOINT_NAME}) ]]; then
    echo "Endpoint for $MODEL_NAME already exists"
else
    echo "Creating Endpoint for $MODEL_NAME"
    gcloud ai endpoints create --region=${REGION} --display-name=${ENDPOINT_NAME}
fi

ENDPOINT_ID=$(gcloud ai endpoints list --region=$REGION \
    --format='value(ENDPOINT_ID)' --filter=display_name=${ENDPOINT_NAME})
echo "ENDPOINT_ID=$ENDPOINT_ID"

# Delete any existing models with this name
for MODEL_ID in $(gcloud ai models list --region=$REGION \
    --format='value(MODEL_ID)' --filter=display_name=${MODEL_NAME}); do
    echo "Deleting existing $MODEL_NAME ... $MODEL_ID"
    gcloud ai models delete --region=$REGION $MODEL_ID
done

# Upload the model
gcloud ai models upload --region=$REGION --display-name=$MODEL_NAME \
    --container-image-uri=us-docker.pkg.dev/vertex-ai/prediction/tf2-cpu.${TF_VERSION}:latest \
    --artifact-uri=$EXPORT_PATH

MODEL_ID=$(gcloud ai models list --region=$REGION \
    --format='value(MODEL_ID)' --filter=display_name=${MODEL_NAME})
echo "MODEL_ID=$MODEL_ID"

# Deploy the model to the endpoint
gcloud ai endpoints deploy-model $ENDPOINT_ID \
    --region=$REGION \
    --model=$MODEL_ID \
    --display-name=$MODEL_NAME \
    --machine-type=e2-standard-2 \
    --min-replica-count=1 \
    --max-replica-count=1 \
    --traffic-split=0=100
```

**Notes:**

- If you encounter a model building error (e.g., service account permissions), try running the code cell again
- Enable the Vertex AI API if not enabled
- This process will take around 15-20 minutes
- To see progress, navigate to **Navigation menu > Vertex AI > Endpoints** in the Cloud Console

✅ **Check my progress:** Deploy flights model to Vertex AI

---

### Test the Model Endpoint

2. Create a test input file `example_input.json`:

```python
%%writefile example_input.json
{"instances": [
    {"dep_hour": 2, "is_weekday": 1, "dep_delay": 40, "taxi_out": 17, "distance": 41, "carrier": "AS", "dep_airport_lat": 58.42527778, "dep_airport_lon": -135.7075, "arr_airport_lat": 58.35472222, "arr_airport_lon": -134.57472222, "origin": "GST", "dest": "JNU"},
    {"dep_hour": 22, "is_weekday": 0, "dep_delay": -7, "taxi_out": 7, "distance": 201, "carrier": "HA", "dep_airport_lat": 21.97611111, "dep_airport_lon": -159.33888889, "arr_airport_lat": 20.89861111, "arr_airport_lon": -156.43055556, "origin": "LIH", "dest": "OGG"}
]}
```

3. Make a prediction from the model endpoint:

```bash
%%bash
ENDPOINT_ID=$(gcloud ai endpoints list --region=$REGION \
    --format='value(ENDPOINT_ID)' --filter=display_name=${ENDPOINT_NAME})
echo $ENDPOINT_ID
gcloud ai endpoints predict $ENDPOINT_ID --region=$REGION --json-request=example_input.json
```

4. Alternatively, send an HTTP POST request:

```bash
%%bash
PROJECT=$(gcloud config get-value project)
ENDPOINT_ID=$(gcloud ai endpoints list --region=$REGION \
    --format='value(ENDPOINT_ID)' --filter=display_name=${ENDPOINT_NAME})

curl -X POST \
    -H "Authorization: Bearer $(gcloud auth application-default print-access-token)" \
    -H "Content-Type: application/json; charset=utf-8" \
    -d @example_input.json \
    "https://${REGION}-aiplatform.googleapis.com/v1/projects/${PROJECT}/locations/${REGION}/endpoints/${ENDPOINT_ID}:predict"
```

---

## Task 6. Model Explainability

Model explainability is one of the most important problems in machine learning. It's a broad concept of analyzing and understanding the results provided by machine learning models. **Explainable AI (XAI)** makes models transparent and solves the black box problem.

### Inspect Model Signature

1. View the model's input and output signature:

```bash
%%bash
model_dir=$(gsutil ls ${OUTDIR}/export | tail -1)
echo $model_dir
saved_model_cli show --tag_set serve --signature_def serving_default --dir $model_dir
```

### Create Explanation Metadata

2. Create a JSON file `explanation-metadata.json` that describes the model's input and output for explanation:

```python
cols = (
    'dep_delay,taxi_out,distance,dep_hour,is_weekday,'
    'dep_airport_lat,dep_airport_lon,'
    'arr_airport_lat,arr_airport_lon,'
    'carrier,origin,dest'
)

inputs = {
    x: {"inputTensorName": f"{x}"}
    for x in cols.split(',')
}

expl = {
    "inputs": inputs,
    "outputs": {
        "pred": {
            "outputTensorName": "pred"
        }
    }
}

print(expl)
with open('explanation-metadata.json', 'w') as ofp:
    json.dump(expl, ofp, indent=2)
```

3. View the metadata file:

```bash
!cat explanation-metadata.json
```

![Metadata](https://cdn.qwiklabs.com/RnBtk9tbdBoyOdXafvlYMed1IlQByADXidS%2FpEZxlSI%3D)

---

### Create and Deploy Explainable Model

4. Create the model endpoint `flights_xai`, upload the model with explainability, and deploy it:

```bash
%%bash

# Change endpoint name for explainable model
ENDPOINT_NAME=flights_xai

TIMESTAMP=$(date +%Y%m%d-%H%M%S)
MODEL_NAME=${ENDPOINT_NAME}-${TIMESTAMP}
EXPORT_PATH=$(gsutil ls ${OUTDIR}/export | tail -1)
echo $EXPORT_PATH

# Create the model endpoint for deploying the model
if [[ $(gcloud ai endpoints list --region=$REGION \
        --format='value(DISPLAY_NAME)' --filter=display_name=${ENDPOINT_NAME}) ]]; then
    echo "Endpoint for $MODEL_NAME already exists"
else
    echo "Creating Endpoint for $MODEL_NAME"
    gcloud ai endpoints create --region=${REGION} --display-name=${ENDPOINT_NAME}
fi

ENDPOINT_ID=$(gcloud ai endpoints list --region=$REGION \
    --format='value(ENDPOINT_ID)' --filter=display_name=${ENDPOINT_NAME})
echo "ENDPOINT_ID=$ENDPOINT_ID"

# Delete any existing models with this name
for MODEL_ID in $(gcloud ai models list --region=$REGION \
    --format='value(MODEL_ID)' --filter=display_name=${MODEL_NAME}); do
    echo "Deleting existing $MODEL_NAME ... $MODEL_ID"
    gcloud ai models delete --region=$REGION $MODEL_ID
done

# Upload the model with explainability
# Using sampled-shapley method with 10 feature permutations
gcloud ai models upload --region=$REGION --display-name=$MODEL_NAME \
    --container-image-uri=us-docker.pkg.dev/vertex-ai/prediction/tf2-cpu.${TF_VERSION}:latest \
    --artifact-uri=$EXPORT_PATH \
    --explanation-method=sampled-shapley \
    --explanation-path-count=10 \
    --explanation-metadata-file=explanation-metadata.json

MODEL_ID=$(gcloud ai models list --region=$REGION \
    --format='value(MODEL_ID)' --filter=display_name=${MODEL_NAME})
echo "MODEL_ID=$MODEL_ID"

# Deploy the model to the endpoint
gcloud ai endpoints deploy-model $ENDPOINT_ID \
    --region=$REGION \
    --model=$MODEL_ID \
    --display-name=$MODEL_NAME \
    --machine-type=e2-standard-2 \
    --min-replica-count=1 \
    --max-replica-count=1 \
    --traffic-split=0=100
```

**Note:** This will take around 15-20 minutes to complete. To see progress, navigate to **Navigation menu > Vertex AI > Endpoints** in the Cloud Console.

✅ **Check my progress:** Deploy flights_xai model to Vertex AI

---

## Task 7. Invoke the Deployed Model

Now that you have deployed a model with explainability, you can invoke it to get predictions along with explanations.

### Get Predictions with Explanations

Run the following code to get predictions with explanations:

```bash
%%bash
PROJECT=$(gcloud config get-value project)
ENDPOINT_NAME=flights_xai
ENDPOINT_ID=$(gcloud ai endpoints list --region=$REGION \
    --format='value(ENDPOINT_ID)' --filter=display_name=${ENDPOINT_NAME})

curl -X POST \
    -H "Authorization: Bearer $(gcloud auth application-default print-access-token)" \
    -H "Content-Type: application/json; charset=utf-8" \
    -d @example_input.json \
    "https://${REGION}-aiplatform.googleapis.com/v1/projects/${PROJECT}/locations/${REGION}/endpoints/${ENDPOINT_ID}:explain"
```

The response will include both predictions and feature attributions that explain how each feature contributed to the prediction.

---

## Congratulations! 🎉

**Congratulations!** In this lab, you learned how to:

- Create a TensorFlow model using Vertex AI Workbench
- Train a wide-and-deep neural network model
- Deploy models to Vertex AI Endpoints
- Use Vertex AI Explainable AI (XAI) to explain model predictions
- Invoke deployed models via API

You successfully trained a logistic regression model with feature engineering techniques like bucketing, crossing, and embedding to predict flight arrival delays.

---

### Cleanup (Optional)

To avoid incurring charges to your Google Cloud account, consider deleting the resources created in this lab:

```bash
# Delete endpoints
gcloud ai endpoints delete ENDPOINT_ID --region=REGION

# Delete models
gcloud ai models delete MODEL_ID --region=REGION

# Delete Cloud Storage bucket
gsutil rm -r gs://BUCKET_NAME
```

---

### What's Next?

- Explore [Vertex AI documentation](https://cloud.google.com/vertex-ai/docs)
- Learn more about [TensorFlow](https://www.tensorflow.org/)
- Read _[Data Science on the Google Cloud Platform, 2nd Edition](https://www.oreilly.com/library/view/data-science-on/9781098118945/)_
