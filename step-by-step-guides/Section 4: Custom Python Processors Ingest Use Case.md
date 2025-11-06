# Section 3 - Custom Python Processors Ingest Use Case

## Description
Using a custom Python processor in NiFi 2, enabling a developer to add functionality not provided by the standard NiFi processors but still running and managing it as part of a larger flow.

### Lab 1: Deploy a NiFi Flow from the Catalog

- Open the Catalog
- Search for `Telco Usecase 3 Custom Processors Participant`
- Click on `Telco Usecase 3 Custom Processors Participant`
- Click `Deploy`
- Select the HoL Workspace for the HoL (there should only be one; if not, ask the Instructor) then click `Continue`
- Populate the Overview Page:
  - Enter a Distinct Deployment Name using your User ID or Name, e.g. Usecase 3 Python [your initials]
  - Leave the rest of the fields blank.
  - Click `Next`

![alt text](/img/image102.png)

- On the `NiFi Configuration` page:
  - Check the box for Custom Python Configuration (`This flow deployment uses custom python processors`)
  - Add your username and workload password
  - Add the path to the custom processors: `jt-demo-1-buk-046d9e83/data/nifi-custom-processors/`


![alt text](/img/image103.png)

- Populate the Parameters Page with the API-KEY, which your instructor will provide, then click `Next`.

![alt text](/img/image104.png)

- Click `Next` on the `Sizing & Scaling Page`.
- Click `Next` on the `Key Performance Indicators Page`.
- Click `Deploy` on the `Review Page`.
- Click anywhere on the screen. You have now navigated to the `Deployments` page.
- Search for your Deployment if it is not visible on the main page.

### Lab 2: Navigating to your Deployment and Opening the Canvas

- Click on your Deployment.
- Click on `Actions` on the Top Right and select `View in NiFi` to open the Canvas.
- Double-click on the Processor in the middle (ignore any errors).

![alt text](/img/image105.png)

- Stop the `Prompt` component, so we can modify it and run it a few times.

![alt text](/img/image106.png)

- Double-click on the `Prompt` processor and click on the `Properties` tab.

![alt text](/img/image107.png)

- Change the Custom text field to a recipe you would like to generate, click `Apply`
- Right-click on `Prompt` and `Run Once`
- Check that the `success` queue gets a single record, which is processed quickly and moves to the `success` queue after `RecipeGenerator`
- Right-click the output queue and click `List Queue`

![alt text](/img/image108.png)

- In the queue list, click `Download content` to download the generated recipe.

![alt text](/img/image109.png)

- Click `Back to Connection` to return to the canvas.  Before running the flow again, let's stop the RecipeGenerator so we can check the input as well as the output.  Right-click `RecipeGenerator` and click `Stop`.
- If you want, change the recipe prompt (see the earlier steps for guidance) then click `Run Once` - this creates a single FlowFile for our process, but otherwise leaves the `Prompt` processor in a `Stopped` state.
- Check there is a single record in the queue for the RecipeGenerator:

![alt text](/img/image110.png)

- Right-click and choose `List queue` to see the contents
- Go back to the canvas and click `Run Once` on `RecipeGenerator` to process that single record
- Check the output queue now shows the additional recipe.

