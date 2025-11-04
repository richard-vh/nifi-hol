# Section 3 - Custom Python Processors Ingest Use Case

## Description
Using a custom Python processor in NiFi 2, enabling a developer to add functionality not provided by the standard NiFi processors but still running and managing it as part of a larger flow.

### Lab 1: Deploy a NiFi Flow from the Catalog

- Open the Catalog
- Search for "Telco Usecase 3 Custom Processors Participant"
- Click on "Telco Usecase 3 Custom Processors Participant"
- Click "Deploy"
- Select the HoL Workspace for the HoL (there should only be one; if not, ask the Instructor) then click "Continue"
- Populate the Overview Page:
  - Enter a Distinct Deployment Name using your User ID or Name", e.g. Usecase 3 Python [your initials]
  - Leave the rest of the fields blank.
  - Click "Next"

![alt text](/img/image102.png)

- On the "NiFi Configuration" page:
  - Check the box for Custom Python Configuration (“This flow deployment uses custom python processors”)
  - Add your username and workload password
  - Add the path to the custom processors: jt-demo-1

![alt text](/img/image103.png)

- Populate the Parameters Page with the API-KEY, which your instructor will provide, then click ‘Next’.
