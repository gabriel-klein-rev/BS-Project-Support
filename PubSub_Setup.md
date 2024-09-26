# Setting Up a Data Pipeline with Pub/Sub

This tutorial will guide you through the setup process for initializing a Cloud Pub/Sub topic, creating a publisher on Cloud Run Functions to push messages to the topic, and creating a subscriber to take messages from the topic and store data in a Bigquery table.

### Step 1: Set Up Your GCP Environment

#### 1.1: Create a GCP Project
If you haven’t already created a GCP project, do so:
- Navigate to the [GCP Console](https://console.cloud.google.com/).
- Create a new project.

#### 1.2: Enable the Required APIs
Enable the following APIs for your project:
- **Cloud Pub/Sub API**
- **BigQuery API**
- **Cloud Functions API**


### Step 2: Create the Pub/Sub Topic

1. **Create a Pub/Sub topic**:
   - Go to **Pub/Sub** > **Topics**.
   - Click **Create Topic**, name it (e.g., `my-topic`), and click **Create**.

### Step 3: Build the Publisher (Cloud Function)

We’ll create a Cloud Function that acts as the publisher and sends JSON data to the Pub/Sub topic.

#### 3.1: Python Script for the Publisher

Here’s an example Python script that publishes messages to Pub/Sub in JSON format:

```python
import json
import base64
from google.cloud import pubsub_v1
import os

# Set your project ID and topic ID
project_id = "{project_id}" # Replace with your project ID
topic_id = "my-topic"

# Initialize Pub/Sub publisher client
publisher = pubsub_v1.PublisherClient()
topic_path = publisher.topic_path(project_id, topic_id)

def publish_message(request):
    """HTTP Cloud Function to publish a message to Pub/Sub."""
    request_json = request.get_json()

    # Create the message data (in JSON format)
    message_data = json.dumps(request_json).encode('utf-8')

    # Publish the message
    future = publisher.publish(topic_path, message_data)
    future.result()  # Wait for the publish to succeed

    return 'Message published successfully!'
```

#### 3.2: Setup the requirements.txt

Here’s how your requirements.txt might look:

```
google-cloud-pubsub==2.*  # or the latest stable version
```

#### 3.3: Deploy the Publisher Function

1. In your GCP Console, navigate to **Cloud Functions** > **Create Function**.
2. Set the following configuration:
   - **Function name**: `pubsub-publisher`
   - **Runtime**: Python 3.12 (or whichever version you're using)
   - **Trigger**: HTTPS
3. Upload the Python script (or use the in-line editor), set entry point to `publish_message`.
4. Click **Deploy**.

### Step 4: Build the Subscriber (Cloud Function)

This Cloud Function will be triggered when a message is published to Pub/Sub. It will receive the message, validate it, and store it in BigQuery.

#### 4.1: Create a BigQuery Dataset and Table
1. Go to **BigQuery** > **Create Dataset**.
2. Create a new dataset (e.g., `my_dataset`).
3. Create a table in that dataset (e.g., `my_table`) with the schema matching the data format (for this case, we'll be using "name" and "age" for our data).

#### 4.2: Python Script for the Subscriber

This function will receive the JSON data, validate it, and insert it into BigQuery:

```python
import json
import base64
from google.cloud import bigquery
from google.cloud import pubsub_v1

# Initialize the BigQuery client
client = bigquery.Client()

def process_pubsub_event(event, context=""):
    """Triggered from a message on a Cloud Pub/Sub topic."""
    # Decode the Pub/Sub message
    if 'data' in event:
        pubsub_message = base64.b64decode(event['data']).decode('utf-8')
        data = json.loads(pubsub_message)
        
        # Perform data validation (simple example: check for required keys)
        if 'name' in data and 'age' in data:
            insert_row_into_bigquery(data)
        else:
            print('Invalid data received:', data)

def insert_row_into_bigquery(data):
    """Insert a row of data into the BigQuery table."""
    table_id = 'your-project-id.my_dataset.my_table'  # Set your project and table ID

    # Insert row into BigQuery
    errors = client.insert_rows_json(
        table_id, [data]
    )

    if errors == []:
        print('New row inserted into BigQuery.')
    else:
        print('Errors occurred while inserting:', errors)
```

#### 4.3: Setup the requirements.txt

Here’s how your requirements.txt might look:

```
google-cloud-pubsub==2.*  # or the latest stable version
google-cloud-bigquery==3.*
```

#### 4.4: Deploy the Subscriber Function

1. In your GCP Console, navigate to **Cloud Functions** > **Create Function**.
2. Set the following configuration:
   - **Function name**: `pubsub-subscriber`
   - **Runtime**: Python 3.12 (or whichever version you're using)
   - **Trigger**: Pub/Sub
   - **Pub/Sub topic**: Select the topic created in step 2 (`my-topic`).
3. Upload the Python script (or use in-line editor), set entry point to `process_pubsub_event`.
4. Set the **Service Account** to the one created in step 1.3.
5. Click **Deploy**.

### Step 5: Testing the Pipeline

Now that the pipeline is set up, you can test it by triggering the publisher function and checking if the subscriber function processes the message and stores it in BigQuery.

#### 5.1: Test the Publisher
1. Go to the Cloud Functions section in your GCP Console.
2. Trigger the **pubsub-publisher** function by making an HTTPS POST request to it. You can do this via the **Testing tab** or with a tool like **Postman** or **curl**.

Example `curl` request:

```bash
curl -X POST https://<your-publisher-url> \
-H "Content-Type: application/json" \
-d '{"name": "John Doe", "age": 30}'
```

#### 5.2: Verify Data in BigQuery
1. After triggering the publisher, the subscriber will be invoked.
2. Go to **BigQuery** > **my_table** and query the table to verify the data was inserted.

### Step 6: Managing Credentials

For the Python scripts running in Cloud Functions:
1. You don’t need to manually specify credentials if you’ve set up the service accounts correctly for the Cloud Functions.
2. The **Google Cloud Client Libraries** automatically use the credentials assigned to the function’s service account.


You must assign the Invoker role (roles/run.invoker) through Cloud Run if you want to allow the function to receive requests from additional principals or other given authorities in IAM.

### Conclusion

You now have a working pipeline that:
1. Publishes JSON data to Pub/Sub from a Cloud Function.
2. Subscribes to the Pub/Sub topic, validates the data, and stores it in BigQuery via another Cloud Function.
