# Section 2 - Evaluating Quality of Service on Ingest

## Description
Add some content!!!!!

## Pre-Requisites

### Stage 1: Pre-Requisites

- Create Iceberg Table for NiFi Flow Exceptions:

![alt text](/img/image1.png)

```ruby
CREATE TABLE <USER_ID>_QOS_POOR_FAILURE_ICEBERG (
wifi_band STRING,
bssid STRING,
connection_status STRING,
data_usage_mb STRING,
device_ip STRING,
current_session_duration_seconds STRING,
device_mac STRING,
time_since_first_connected_seconds STRING,
first_connected STRING,
mac_oui STRING,
`timestamp` STRING,
signal_strength_dbm STRING
)
STORED AS ICEBERG;
```

## NiFi Development

### Stage 2: Deploy a NiFi Flow from the Catalog

- Open the Catalog.


![alt text](/img/image2.png)

- Search for "Telco Usecase 1- Quality of Service on Ingest - Participant".

![alt text](/img/image3.png)

- Click on "Telco Usecase 1- Quality of Service on Ingest - Participant 2".

![alt text](/img/image4.png)
![alt text](/img/image5.png)

- Click "Deploy".

![alt text](/img/image6.png)

- Select the HoL Workspace for the HoL (there should only be one; if not, ask the Instructor) then click "Continue".

![alt text](/img/image7.png)

- **Populate the Overview Page:**
  - Enter a "Distinct Deployment Name using your User ID or Name".
  - Leave the rest of the fields blank.
  - Click "Next".
 
![alt text](/img/image8.png)

- Click "Next" on the "NiFi Configuration" Page.

![alt text](/img/image9.png)

- Populate the Parameters Page:
  - Enter your Workload Username.
  - Enter your Workload Password.
  - Click "Next".

![alt text](/img/image10.png)

- Click "Next" on the "Sizing & Scaling Page".

![alt text](/img/image11.png)

- Click "Next" on the "Key Performance Indicators Page".

![alt text](/img/image12.png)

- Click "Deploy" on the "Review Page".

![alt text](/img/image13.png)

- Click anywhere on the screen. You have now navigated to the "Deployments" page. Search for your Deployment if it is not visible on the main page.

![alt text](/img/image14.png)

### Stage 3: Navigating to your Deployment and Opening the Canvas

- Click on your Deployment.

![alt text](/img/image15.png)
![alt text](/img/image16.png)

- Click on "Actions" on the Top Right and select "View in NiFi" to open the Canvas.

![alt text](/img/image17.png)
![alt text](/img/image18.png)

### Stage 4: Building on the Canvas

- Double-click on the Processor in the middle (ignore any errors).

![alt text](/img/image19.png)
![alt text](/img/image20.png)


- You should see three Processors deployed within your existing Canvas (click and drag the screen to navigate the Canvas).

![alt text](/img/image21.png)

- Right-click on the "Execute Script" Processor and select "Configure".
  - **View Settings:** Here you can give the Processor a custom name.
  - **View Scheduling:** Here you can decide how and how often it gets executed.
  - **View Properties:** Single-click the "Script Body" to see what will be run.
  - **View Relationships:** Note that the "failure" relationship has been terminated.

![alt text](/img/image22.png)

- Drag a new "Processor" onto the Canvas.
  - Hold click on the Processor Icon.

![alt text](/img/image23.png)

- Drag onto the Canvas and release.

![alt text](/img/image24.png)
![alt text](/img/image25.png)

- Search for the "SplitJson" Processor and select "ADD".

![alt text](/img/image26.png)
![alt text](/img/image27.png)

- Once you add the Processor, you can click and hold it to move it to a new location.

![alt text](/img/image28.png)

- Hover over the "Execute Script" Processor and note the downward arrow.

![alt text](/img/image29.png)

- Click on the downward arrow, attach it to the "SplitJson" processor, create the connector for the "Success" Relationship, and click "ADD".

![alt text](/img/image30.png)

- Right-click on the "SplitJson" Processor and select "Configure".
  - Navigate to "Properties".
  - Populate the "JsonPath Expression" value as \$\[\*\]
  - Click "Apply".
 
![alt text](/img/image31.png)

- Right-click on the "SplitJson" Processor and select "Configure".
  - Navigate to "Relationships"
  - Tick the "Terminate" box for "Failure" and "Original".
  - Click "Apply".

![alt text](/img/image32.png)

- Now add a new Processor called "EvaluateJsonPath" and then "ADD".

![alt text](/img/image33.png)

- Create a new Relationship between "SplitJson" and the "EvaluateJsonPath" processor for the "split" relationship.

![alt text](/img/image34.png)

- Right-click on the "EvaluateJsonPath" and open "Configure".
  - **Within Relationships:**
    - Terminate the "failure" and the "unmatched relationship".

![alt text](/img/image35.png)

- 1. **Within Properties:**
        - Change "destination" to "flowfile-attribute".
        - Add new parameters by clicking the "+" icon:
            - Property Name = `signal_strength_dbm`, Property Value = `$.signal_strength_dbm`
            - Property Name = `wifi_band`, Property Value = `$.wifi_band`

![alt text](/img/image36.png)

- Now add a new Processor called "RouteOnAttribute" and then "ADD".

![alt text](/img/image37.png)

- Create a new Relationship from "EvaluateJsonPath" for "Matched".

![alt text](/img/image38.png)

- Right-click on the "RouteOnAttribute" and open "Configure".
  - **Within Relationships:**
    - Terminate for "Unmatched".

![alt text](/img/image39.png)

  - **Within Properties:**
     - Add new parameters by clicking the "+" icon (make sure not to include any trailing whitespace when you copy/paste):
     
       - Property Name1 = `QOS_EXCELLENT`, Property Value =
       
        ```
        ${
        ${wifi_band:equalsIgnoreCase('5 GHz')
        :and(\${signal_strength_dbm:isEmpty():not()
        :and(\${signal_strength_dbm:toNumber():ge(-65)})})}
        :or(
        ${wifi_band:equalsIgnoreCase('2.4 GHz')
        :and(\${signal_strength_dbm:isEmpty():not()
        :and(\${signal_strength_dbm:toNumber():ge(-55)})})}
        )
        }
        ```
        - Property Name = `QOS_FAIR`, Property Value =
          
         ```
         ${
         ${wifi_band:equalsIgnoreCase('5 GHz')
         :and(\${signal_strength_dbm:isEmpty():not()
         :and(\${signal_strength_dbm:toNumber():ge(-75)})
         :and(\${signal_strength_dbm:toNumber():lt(-65)})})}
         :or(
         ${wifi_band:equalsIgnoreCase('2.4 GHz')
         :and(\${signal_strength_dbm:isEmpty():not()
         :and(\${signal_strength_dbm:toNumber():ge(-70)})
         :and(\${signal_strength_dbm:toNumber():lt(-55)})})}
         )
         }
         ```
         - Property Name = `QOS_POOR`, Property Value =
           
         ```
         ${signal_strength_dbm:isEmpty():not()
         :and(\${signal_strength_dbm:toNumber():lt(-75)})}
         ```

![alt text](/img/image40.png)

- **Adjust the Kafka Producer Publisher called "PublishKafka2RecordCDP":**
  - Establish a relationship between the "RouteOnAttribute" processor and the "QOS_EXCELLENT - PublishKafka2RecordCDP" Processor for the "QOS_EXCELLENT" relationship.
 
![alt text](/img/image41.png)

- 1. **Within the Properties Tab:**
        - Rename the Topic Name to your own Username Prefix.
    
![alt text](/img/image42.png)

- **Configure remaining relationships for "QOS_FAIR" and "QOS_POOR":**
  - Copy the existing Kafka Processor by right-clicking and selecting "copy".

![alt text](/img/image43.png)

- 1. Right-click on the Canvas and select "Paste" twice to create two duplicates.

![alt text](/img/image44.png)
![alt text](/img/image45.png)

- 1. Configure these additional processors:
        - Processor 2 (QOS_FAIR):
            - Settings: Prefix the Processor with the new name "QOS_FAIR - ".
            - Properties: Topic Name = `JT_QOS_FAIR_KAFKA`.
        - Processor 3 (QOS_POOR):
            - Settings: Prefix the Processor with the new name "QOS_POOR - ".
            - Properties: Topic Name = `JT_QOS_POOR_KAFKA`.

- Establish the relevant relationships between the various processors.

![alt text](/img/image46.png)

- **Implement failover logic for "QOS_POOR" records:**
  - Establish a relationship between "QOS_POOR - PublishKafka2RecordCDP" and the "PutIceberg" Processor for "Failure Relationship". This ensures that "QOS_POOR" records are captured in the event of Kafka cluster unavailability and can be replayed later.

![alt text](/img/image47.png)
![alt text](/img/image48.png)

- Edit the PutIceberg Processor:
  - Within the Properties:
    - Adjust the Table name to what was created during the Pre-Requisites.
    - If you're unable to change the properties, check that the processor is stopped first - click "Stop and Configure" to edit

![alt text](/img/image49.png)

### Stage 5: Running the NiFi Canvas and Viewing the Output

- **Execute the Script:** Right-click the "ExecuteScript" Processor and select "Run Once."

![alt text](/img/image50.png)

- **Verify Queued Message:** Observe that the "success" relationship now indicates one queued message.

![alt text](/img/image51.png)

- **List the Queue:** Right-click on the relationship with the queued message and select "List Queue."

![alt text](/img/image52.png)

- **View Flow File:** A new page should open, displaying the Flow File created in the previous step.

![alt text](/img/image53.png)

- **Examine Message Content:** Click the "Eye" icon on the right-hand side to view the message.

![alt text](/img/image54.png)

- **Review JSON String:** A new tab will open, presenting a large JSON string.

![alt text](/img/image55.png)

- **Continue Flow and Analysis:** Close the tab and proceed to pass the message through the rest of the flow, examining how the output changes at each stage. You can do this by clicking "Run Once" on each component in turn.
- **Run the Entire Canvas:** Once your analysis is complete, right-click on the canvas and select "Start" This will execute the entire canvas continuously until it is manually stopped. (Please stop the Canvas before you close the window: you do not need to leave it running)

![alt text](/img/image56.png)

## Analysing Kafka via Streams Messaging Manager

### Stage 6: Access Streams Messaging Manager

- - Navigate to the Control Page Home Page.
 
![alt text](/img/image57png)
![alt text](/img/image58.png)

- - Go to the "Management Console."
 
![alt text](/img/image59.png)

- - Select the "Datahubs" Page.
 
![alt text](/img/image60.png)

- - Search for "cdfv1-smm-dh" and click on the Datahub.
 
![alt text](/img/image61.png)
![alt text](/img/image62.png)

- - Click on "Streams Messaging Manager."

![alt text](/img/image63.png)

### Stage 7: Explore Topics

- - Click on "Topics."

![alt text](/img/image64.png)

- - Search for your UserID (these are the topics you previously produced to) and select one of them.

![alt text](/img/image65.png)

### Stage 8: Review Topic Details

- - Click on the "Details" Tab.

![alt text](/img/image66.png)

- - Explore the various tabs at the top of the screen, paying close attention to the "Data Explorer" tab to understand the data being written to the topic.

![alt text](/img/image67.png)
![alt text](/img/image68.png)

## Interacting with Kafka via Flink SQL

### Stage 9: To access the Flink Streaming SQL Console

- Navigate to the Control Page Home Page.
- Go to the "Management Console."
- Select the "Datahubs" Page.
- Search for "cdf-streaming-analytics-dh" and click on the Datahub.

![alt text](/img/image69.png)
![alt text](/img/image70.png)

- Click on "Streaming SQL Console."

![alt text](/img/image71.png)
![alt text](/img/image72.png)

### Stage 10: Keytab Unlock (If prompted)

- Click on your user's icon/name at the bottom-left of the screen.

![alt text](/img/image73.png)

- Select "Manage Keytab."
- Enter your username and workload password, then click "Unlock Keytab."

![alt text](/img/image74.png)

### Stage 11: Project Navigation

- Open the "jturkington_default" project. (Notify the instructor for assistance if you cannot see or access the Project)

![alt text](/img/image75.png)

### Stage 12: Running Flink SQL Jobs

- Click on "Jobs," select "New Job," and provide a name before clicking "Create."

![alt text](/img/image76.png)
![alt text](/img/image77.png)
![alt text](/img/image78.png)

- For this demonstration, I have pre-configured access to my Kafka Topics. You will use the instructor's topics rather than configuring access to your own. (A demo will be provided on how to access your existing Kafka Topics.)
- Copy the following query into the editor and click "execute":

```ruby
SELECT * FROM JT_QOS_POOR_KAFKA
```

![alt text](/img/image79.png)

- Execute the next query to perform a window action, counting and aggregating data by BSSID as it arrives in the Kafka Topic:

```ruby
SELECT *,
       PROCTIME() AS proc_time
FROM `ssb`.`jturkington_default`.`JT_QOS_POOR_KAFKA` )
SELECT window_start,
       window_end,
       bssid,
       COUNT(*) AS record_count,
       SUM(data_usage_mb) AS total_usage_mb,
       AVG(CAST(signal_strength_dbm AS DOUBLE)) AS avg_signal_dbm,
       MIN(signal_strength_dbm) AS min_signal_dbm,
       MAX(signal_strength_dbm) AS max_signal_dbm
FROM TABLE(TUMBLE(TABLE src, DESCRIPTOR(proc_time), INTERVAL '10' SECOND))
GROUP BY window_start,
         window_end,
         bssid;
```

- Finally, run the last query to enrich the data with information from our data lake (a Kudu Table). This query uses the OUI ID to map to a more readable device name, allowing us to understand device counts over time:

```ruby
WITH src AS
  (SELECT *,
          PROCTIME() AS proc_time
   FROM `ssb`.`jturkington_default`.`JT_QOS_POOR_KAFKA`),
     win AS
  (SELECT window_start,
          window_end,
          mac_oui
   FROM TABLE(TUMBLE(TABLE src, DESCRIPTOR(proc_time), INTERVAL '15' SECOND)))
SELECT w.window_start,
       w.window_end,
       COALESCE(d.device_type, 'Unknown') AS device_type,
       COUNT(*) AS total_device_type_count
FROM win AS w
LEFT JOIN `cdf-kudu-dh-kudu`.`default`.`default.device_oui_dim` d ON w.mac_oui = d.mac_oui
GROUP BY w.window_start,
         w.window_end,
         COALESCE(d.device_type, 'Unknown');
```

# Appendix

## Kudu Create Table Statement

```ruby
CREATE TABLE default.JT_device_oui_dim (
mac_oui STRING,
vendor STRING,
device_type STRING,
PRIMARY KEY (mac_oui)
)
PARTITION BY HASH(mac_oui) PARTITIONS 4
STORED AS KUDU;

INSERT INTO default.JT_device_oui_dim VALUES
('00:03:93','Apple','Phone/Tablet/PC'),
('00:0A:27','Apple','Phone/Tablet/PC'),
('00:16:CB','Apple','Phone/Tablet/PC'),
('00:1B:63','Apple','Phone/Tablet/PC'),
('00:1E:52','Apple','Phone/Tablet/PC'),
('00:21:E9','Apple','Phone/Tablet/PC'),
('00:23:DF','Apple','Phone/Tablet/PC'),
('00:25:00','Apple','Phone/Tablet/PC'),
('00:26:08','Apple','Phone/Tablet/PC'),
('00:30:65','Apple','Phone/Tablet/PC'),
('00:50:E4','Apple','Phone/Tablet/PC'),
('00:90:F5','Apple','Phone/Tablet/PC'),
('10:93:E9','Apple','Phone/Tablet/PC'),
('38:F9:D3','Apple','Phone/Tablet/PC'),
('40:A6:B7','Apple','Phone/Tablet/PC'),
('64:BC:0C','Apple','Phone/Tablet/PC'),
('74:E7:5C','Apple','Phone/Tablet/PC'),
('88:1F:A1','Apple','Phone/Tablet/PC'),
('A8:20:66','Apple','Phone/Tablet/PC'),
('AC:E4:B6','Apple','Phone/Tablet/PC'),
('F0:DC:C4','Apple','Phone/Tablet/PC'),
('FC:E9:98','Apple','Phone/Tablet/PC'),
('AC:FD:EC','Apple','Phone/Tablet/PC'),
('00:04:1F','Sony','PlayStation'),
('00:13:15','Sony','PlayStation'),
('00:19:C5','Sony','PlayStation'),
('28:0D:FC','Sony','PlayStation'),
('5C:84:3C','Sony','PlayStation'),
('9C:37:CB','Sony','PlayStation'),
('B4:0A:D8','Sony','PlayStation'),
('D4:F7:D5','Sony','PlayStation'),
('00:03:FF','Microsoft','Xbox/Surface'),
('00:12:5A','Microsoft','Xbox/Surface'),
('00:1D:D8','Microsoft','Xbox/Surface'),
('28:18:78','Microsoft','Xbox/Surface'),
('40:8E:2C','Microsoft','Xbox/Surface'),
('5C:BA:37','Microsoft','Xbox/Surface'),
('70:BC:10','Microsoft','Xbox/Surface'),
('74:E2:8C','Microsoft','Xbox/Surface'),
('84:57:33','Microsoft','Xbox/Surface'),
('98:5F:D3','Microsoft','Xbox/Surface'),
('00:09:BF','Nintendo','Nintendo'),
('00:16:56','Nintendo','Nintendo'),
('00:1A:E9','Nintendo','Nintendo'),
('2C:10:C1','Nintendo','Nintendo'),
('40:D2:8A','Nintendo','Nintendo'),
('7C:BB:8A','Nintendo','Nintendo'),
('00:00:F0','Various','IoT/Networking'),
('00:1A:11','Various','IoT/Networking'),
('18:B4:30','Various','IoT/Networking'),
('3C:5A:B4','Various','IoT/Networking'),
('54:60:09','Various','IoT/Networking'),
('A4:C1:38','Various','IoT/Networking'),
('80:48:9A','Various','IoT/Networking'),
('D8:90:E8','Various','IoT/Networking'),
('00:00:0C','Various','IoT/Networking'),
('5C:A3:9D','Various','IoT/Networking'),
('84:7B:EB','Various','IoT/Networking'),
('E8:9E:B8','Various','IoT/Networking');
```









