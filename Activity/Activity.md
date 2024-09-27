### Activity: Building a GCP Pipeline for Mock Store Order Data

**Objective**:  
Create a pipeline that receives order data in JSON format from a mock store, validates the data, and stores it in a BigQuery table using Google Cloud Pub/Sub and Cloud Functions.

**Scenario**:  
Your mock e-commerce store is generating order data that includes the following information:
- `orderid`: A unique ID for each order.
- `customerid`: The ID of the customer placing the order.
- `productid`: The ID of the product purchased.
- `totprice`: The total price of the order.

You need to set up a system that:
1. Publishes the order data as a message to a **Pub/Sub topic**.
2. Subscribes to that topic using a **Cloud Function** to validate and process the data.
3. Stores the validated data in a **BigQuery** table.

### **Steps**:

#### **1. Set Up the Pub/Sub Topic and Subscription**
- **Create a Pub/Sub topic** named `order-data`.
- **Create a Pub/Sub subscription** linked to this topic.

#### **2. Create a BigQuery Table**
- In BigQuery, create a dataset named `mock_store`.
- Within the dataset, create a table called `orders` with the following schema:
  - `orderid`: STRING
  - `customerid`: STRING
  - `productid`: STRING
  - `totprice`: FLOAT

#### **3. Write the Publisher Function**

Write a Python function that publishes a JSON message to the Pub/Sub topic. The message should contain the following fields:
- `orderid`
- `customerid`
- `productid`
- `totprice`



#### **4. Write the Subscriber Function**

Write a Python Cloud Function that subscribes to the Pub/Sub topic. The function should:
- Validate that the `orderid`, `customerid`, `productid`, and `totprice` fields are present.
- Insert the validated data into the `orders` BigQuery table.

#### **5. Deploy the Functions**
- Deploy the **publisher function** as an HTTP-triggered Cloud Function.
- Deploy the **subscriber function** as a Pub/Sub-triggered Cloud Function linked to the `order-data` topic.

#### **6. Test the Pipeline**
- Use **curl** or **Postman** to trigger the publisher function, sending a sample order data in JSON format. Example JSON payload:
```json
{
    "orderid": "order123",
    "customerid": "cust456",
    "productid": "prod789",
    "totprice": 99.99
}
```
- Verify that the subscriber function processes the message and inserts it into the BigQuery table.

### **Deliverables**:
- A Pub/Sub topic (`order-data`).
- A BigQuery table (`mock_store.orders`) populated with order data.
- Two Cloud Functions (a publisher and a subscriber).

