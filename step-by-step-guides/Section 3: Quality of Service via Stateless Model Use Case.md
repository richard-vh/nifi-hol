# Section 3 - Quality of Service via Stateless Model Use Case

## Description
This lab guides you through building an AI-driven data pipeline in Apache NiFi. You will modify a dataflow to call an external, stateless AI model to perform real-time anomaly detection on telco data. You will use Jolt to format the API request, InvokeHTTP to get a prediction from the model, and then route records identified as 'ANOMALY' to a specific Apache Kafka topic.


### Lab 1: Deploy a NiFi Flow from the Catalog

- Open the Catalog.
- Search for **Telco Usecase 3 - Quality of Service via Stateless Model - Participant**.
- Click on **Telco Usecase 3 - Quality of Service via Stateless Model - Participant**.
- Click **Deploy**.
- Select the HoL Workspace for the HoL (there should only be one; if not, ask the Instructor) then click **Continue**.
- Populate the Overview Page:
  - Enter a **Distinct Deployment Name using your User ID or Name**.
  - Leave the rest of the fields blank.
  - Click **Next**.
- Click **Next** on the **NiFi Configuration** Page.
- Populate the Parameters Page:
  - Enter your Workload Username.
  - Enter your Workload Password.
  - Update the Kafka Topic List to:
    ```ruby
       cdfv1-smm-dh-corebroker1.jt-demo.u5hb-n231.a2.cloudera.site:9093,
       cdfv1-smm-dh-corebroker0.jt-demo.u5hb-n231.a2.cloudera.site:9093,
       cdfv1-smm-dh-corebroker2.jt-demo.u5hb-n231.a2.cloudera.site:9093
    ```
  - Click **Next**.
- Click **Next** on the **Sizing & Scaling Page**.
- Click **Next** on the **Key Performance Indicators Page**.
- Click **Deploy** on the **Review Page**.
- Click anywhere on the screen. You have now navigated to the **Deployments** page. Search for your Deployment if it is not visible on the main page.
- Click on your Deployment.
- Click on **Actions** on the Top Right and select **View in NiFi** to open the Canvas.

### Lab 2: Building on the Canvas

- Double-click on the Process Group in the middle of the canvas (ignore any errors).

![alt text](/img/image80.png)

- Ensure the **ExecuteScript** Processor is in a stopped state.

![alt text](/img/image81.png)

- Create a new **JoltTransformJSON** processor on the canvas. We will use this processor to create a payload which will be sent to our model endpoint deployed within Cloudera AI (Demoed Previously).

![alt text](/img/image82.png)

- Edit the **JoltTransformJSON** Processor via Right Click -> Configure
  - Within Settings
    - Rename Processor to **Generate Payload**.

![alt text](/img/image83.png)

- Within Properties
  - Adjust the Jolt Specification to use the below code (Note the accessKey below will be different so please ask your instructor). 

```ruby
[
  {
    "operation": "shift",
    "spec": {
      // 1. Take every key-value pair from the input data ("*")
      // 2. Place it under a new key named "request", maintaining the original key names (&)
      "*": "request.&"
    }
  },
  {
    "operation": "default",
    "spec": {
      // 1. Add a new field named "accessKey" at the root level.
      // 2. Assign the hardcoded token value to it.
      "accessKey": "<provided by your instructor>"
    }
  }
]
```

![alt text](/img/image84.png)

- Within Relationships
  - Terminate the Failure Relationship
  - Ensure you Apply the changes.

![alt text](/img/image85.png)

- Establish a Relationship between the **SplitJson** and the **Generate Payload** processor for the Split Relationship.

![alt text](/img/image86.png)

- If you notice that your SplitJson is still showing an errored state then please ensure all other relationships with the exception of **Split** are terminated.

![alt text](/img/image87.png)

- Create a new InvokeHTTP Processor on the Canvas, we will use this to POST a request to the Model Endpoint.

![alt text](/img/image88.png)

- Edit the **InvokeHTTP** Processor via Right Click -> Configure
  - Within Settings
    - Rename Processor to `Call Model`.

![alt text](/img/image89.png)

- Within Properties
  - HTTP Method = `POST`
  - HTTP URL = `https://<provided by your instructor>/model`
  - Request Content-Type = `application/json`.

![alt text](/img/image90.png)

- Within Relationships
  - Terminate everything exception the Response Relationship.

![alt text](/img/image91.png)

- Establish a Relationship between the **Generate Payload** and the **Call Model** processor for the **Success** Relationship.

![alt text](/img/image92.png)

- Create a new EvaluateJsonPath Processor on the Canvas, we will use this to easily extract the Specification from the JSON
- Edit the **EvaluateJsonPath** Processor via Right Click -> Configure
  - Within Settings
    - Rename Processor to `ExtractKeyAttributes`.

![alt text](/img/image93.png)

- Within Properties
  - Destination = flowfile-attribute
  - Add a new Parameter `anomaly.classification` with the value `$.response.prediction.results.classification`.

![alt text](/img/image94.png)

- Within Relationships
  - Terminate everything except the **Matched** Relationship.

![alt text](/img/image95.png)

- Establish a Relationship between the **Call Model** and the **ExtractKeyAttributes** processor for the **Response** Relationship.

![alt text](/img/image96.png)

- Create a new RouteOnAttribute Processor on the Canvas, we will use this to route the ANOMALIES and NORMAL Records accordingly.

![alt text](/img/image97.png)

- Edit the **RouteOnAttribute** Processor via Right Click -> Configure
  - Within Properties
    - Add a new Parameter `Anomaly_Alert` with the value `${anomaly.classification:equals('ANOMALY')}`.

![alt text](/img/image98.png)

- Within Relationships
  - Terminate everything except the **Matched** Relationship (if the Anomaly_Alert relationship doesn't show in the **Relationships** tab of the processor configuration, **Apply** the changes made so far and re-open the **Relationships** tab).

![alt text](/img/image99.png)

- Establish a Relationship between the **ExtractKeyAttributes** and the **RouteOnAttribute** processor for the **Matched** Relationship.

![alt text](/img/image100.png)

- Edit the **ANOMALY - PublishKafka2RecordCDP** Processor via Right Click -> Configure
  - Within Properties
    -	Rename the Topic Name using your Username Prefix or a Unique Identified (i.e. User001-ANOMALY_KAFKA or JT-ANOMALY_KAFKA)
  - Within Relationships
    - Terminate all relationships.

![alt text](/img/image101.png)

- Establish a Relationship between the **RouteOnAttribute** and the **ANOMALY - PublishKafka2RecordCDP** processor for the **Anomaly_Alert** Relationship.

### Lab 3: Running the NiFi Canvas and Viewing the Output

As with the prior use cases, use the **Run Once** functionality to view the output at each stage alongside using SSM to view the contents of the Kafka Topic.

Due to the random nature of the data used in this use-case, the first record that flows through the pipeline (as you click **Run Once** on each processor) may return a failure response from the model endpoint - this is expected.  To test properly you can start all the processors except the first and last in the pipeline, then **Run Once** on the first processor (ExecuteScript).  This will generate ~200 records into the pipeline, and you'll see some end up in the queue for the Anomaly Kafka writer processor.



