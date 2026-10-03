# Azure-DevOps-Build-Deploy-YouTube-Clone-hands-on-labs-
Production-oriented Azure DevOps CI/CD hands-on labs covering Classic Pipelines, YAML Pipelines, Azure App Service, npm builds, artifacts, deployments, troubleshooting, and practical DevOps workflows.



# Azure DevOps Build & Deploy — YouTube Clone

## Practical Demos/Labs

**Demo 1 ==> YouTube Clone Repository Setup & Azure App Service Provisioning**

**Demo 2 ==> Classic Azure DevOps CI/CD Pipeline**

**Demo 3 ==> YAML-Based Azure DevOps CI/CD Pipeline**

------- Before moving setup Infra

## Before moving to Demo - Steps to set the infrastructure
- Login to VSCode or any other IDE of your choice
- Run the below commands to download the application code
  ```
  mkdir day4_youtube_clone; cd day4_youtube_clone
  git init
  git clone https://github.com/piyushsachdeva/Youtube_Clone
  ```
- Create a project in Azure DevOps for Day4 and push the code by running the below commands on VSCode:
  ```
  git remote add origin "URL-YOURAZUREREPO"
  git push -u origin all
  ```
  Note: Make sure to update your Azure repo in the above command
- Go to the Azure Portal and Create the Azure App Service by following Steps : 

- Implement the build pipeline using the classic editor

- Understand the use of service connection and service principal

<img width="1008" height="444" alt="image" src="https://github.com/user-attachments/assets/97fe74a3-20eb-4bea-894f-d5f54539b4cc" />


**Note: You must set the app settings WEBSITE_DYNAMIC_CACHE=0 and WEBSITE_LOCAL_CACHE_OPTION=Never to disable all file caching**

## Structure of Azure DevOps build Pipeline

<img width="727" height="441" alt="image" src="https://github.com/user-attachments/assets/4e319794-e2ff-47ea-9202-3c471e0da1fa" />


*  A trigger tells a Pipeline to run. It could be CI or Scheduled, manual(if not specified), or after another build finishes.
*  A pipeline is made up of one or more stages. A pipeline can deploy to one or more environments.
*  A stage organizes jobs in a pipeline, and each stage can have one or more jobs.
*  Each job runs on one agent, such as Ubuntu, Windows, macOS, etc. A job can also be agentless.
*  Each agent runs a job that contains one or more steps.
*  A step can be a task or script and is the smallest building block of a pipeline.
*  A task is a pre-packaged script that performs an action, such as invoking a REST API or publishing a build artifact.
*  An artifact is a collection of files or packages published by a run.

<img width="1030" height="390" alt="image" src="https://github.com/user-attachments/assets/b9402b03-758e-4c48-b2a8-abc9f496bc2c" />


## Pipeline code used in the demo (Change as per the User Specific Name)

``` YAML
trigger:
- main

stages:
- stage: Build
  jobs:
  - job: Build
    pool:
      vmImage: 'ubuntu-latest'
    steps:
    - task: Npm@1
      inputs:
        command: 'install'
    - task: Npm@1
      inputs:
        command: 'custom'
        customCommand: 'run build'
    - task: PublishBuildArtifacts@1
      inputs:
        PathtoPublish: 'build'
        ArtifactName: 'drop'
        publishLocation: 'Container'
- stage: Deploy
  jobs:
  - job: Deploy
    pool:
      vmImage: 'ubuntu-latest'
    steps:
    - task: DownloadBuildArtifacts@1
      inputs:
        buildType: 'current'
        downloadType: 'single'
        artifactName: 'drop'
        downloadPath: '$(System.ArtifactsDirectory)'
    - task: AzureRmWebAppDeployment@4
      inputs:
        ConnectionType: 'AzureRM'
        azureSubscription: 'Account_Name (Account_ID)'
        appType: 'webAppLinux'
        WebAppName: '(Web App Name )'
        packageForLinux: '$(System.ArtifactsDirectory)/drop'
        RuntimeStack: 'STATICSITE|1.0'
```


================================ ===================== 

# MAIN DEMO: YouTube Clone Repository Setup & Azure App Service Provisioning

## Part 1: Clone the YouTube Clone Application

1. Open the GitHub repository containing the YouTube Clone application.

2. Click **Clone** and copy the repository **Clone URL**.

3. Open **Visual Studio Code** or the code editor of your choice.

4. Navigate to the directory where you want to keep the application.

5. Initialize the Git repository:

```bash
git init
```

6. Clone the application repository:

```bash
git clone <Clone URL>
```

7. Change into the cloned repository:

```bash
cd <repository-folder>
```

8. Verify that the application source code is available inside the directory.

9. The application will later use **npm dependencies**, generate a **build**, and be served through **Azure Web App**.

---

## Part 2: Create an Azure DevOps Project

1. Open:

```text
https://dev.azure.com
```

2. Create a new Azure DevOps project.

3. Give the project the name:

```text
YouTube Clone
```

4. Create the project. (By selecting Private)
<img width="1365" height="646" alt="image" src="https://github.com/user-attachments/assets/5e9a5749-9751-4360-80d5-ea2873355ad8" />

5. Open **Repos**.

6. Use the default repository created for the project.

---

## Part 3: Push the Existing Application to Azure Repos

1. Open the Azure Repos repository.

2. Select the option for pushing an existing repository from the command line.

3. Copy the provided commands.

4. Return to the terminal and make sure you are inside the application directory.

5. Check the existing Git remote:

```bash
git remote -v
```

6. If the existing `origin` points to the original GitHub repository, remove it:

```bash
git remote remove origin
```

7. Add the Azure Repos URL as the new `origin`:   (Remember - We require the Azure repo origin NOT the Github Origin )

```bash
git remote add origin <Azure-Repos-URL>
```

8. Push the application code to Azure Repos:

```bash
git push -u origin <branch-name>
```
<img width="1365" height="647" alt="image" src="https://github.com/user-attachments/assets/d83d8946-b63c-4716-b84a-4008dc3ede82" />

9. Return to **Azure DevOps → Repos**.

10. Refresh the repository and verify that the application source code has been uploaded.

==> **Additional Information:** The (Demo step stated) demonstrates that the existing `origin` initially pointed to GitHub. The remote had to be removed and recreated so that Azure Repos became the destination before pushing the application.

---

## Part 4: Provision Azure App Service

1. Open the Azure Portal:

```text
https://portal.azure.com
```

2. Search for **App Services**.

3. Select **Create → Web App**.

4. Configure the basic settings.

### Subscription

Select the required Azure subscription.

### Resource Group

Create or select a resource group.

The (Demo step stated) uses:

```text
Wed_Rg
```

### Web App Name

Provide a globally unique Web App name.

The (Demo step stated) uses a name similar to:

```text
VyankYouTubeClone
```

The application receives an Azure domain similar to:

```text
<app-name>.azurewebsites.net
```

==> **Additional Information:** The Web App name must be unique because Azure provides a DNS name for the application.

---

## Part 5: Configure the Web App Runtime

1. For **Publish**, select:

```text
Code
```

2. Select the runtime stack:

```text
Node 18
```

3. Select the operating system:

```text
Linux
```

4. Select the required Azure region.

The (Demo step stated) uses:

```text
Canada Central
```

5. Configure the **App Service Plan**.

6. If required, create a new App Service Plan.

The (Demo step stated) uses:

```text
ASP day 4
```
<img width="767" height="1240" alt="image" src="https://github.com/user-attachments/assets/482431f2-b942-4457-a80c-d08921d71aba" />

7. Open **Explore pricing plans**.
<img width="767" height="1240" alt="image" src="https://github.com/user-attachments/assets/3766a3dc-36b0-436e-bac6-091df56f4b0e" />

8. Select the pricing tier required for the demonstration.

The (Demo step stated) uses:

```text
Premium V4
```
<img width="1365" height="650" alt="image" src="https://github.com/user-attachments/assets/59300926-b312-4ed9-9562-90630dc36279" />

9. Keep **Zone redundancy** disabled for this demonstration.

10. Select **Next: Database**.

11. No database is required for this application.

12. Select **Next: Deployment**.

13. Keep continuous deployment disabled because Azure DevOps will be configured separately.

14. Select **Next: Networking**.
<img width="831" height="609" alt="image" src="https://github.com/user-attachments/assets/290962a1-cafb-45f5-ac6e-18207b5ddca3" />

15. Keep **Public Access** enabled.

16. Continue to **Monitoring**.

17. Keep Application Insights disabled for this demonstration.
<img width="695" height="604" alt="image" src="https://github.com/user-attachments/assets/9323b9b3-cd75-4be8-a4fd-7b631f3b378b" />
18. Continue to **Tags**.

19. Select **Review + create**.
<img width="766" height="1246" alt="image" src="https://github.com/user-attachments/assets/2ffad5a8-8c04-4dd1-9b9e-611297e539f3" />

20. Wait for validation to complete.
<img width="681" height="602" alt="image" src="https://github.com/user-attachments/assets/8fc8a3f3-ebd9-43f4-9d1d-aee5d68b5e5e" />

21. Select **Create**.

22. After deployment completes, select **Go to Resource**.

23. Open **Browse** or the default application URL.

24. Verify that the Azure default Web App page is displayed.

==> **Additional Information:** At this point, the infrastructure is provisioned, but the application code has not yet been deployed, so the Azure default page is expected.

---

# MAIN DEMO: Classic Azure DevOps CI/CD Pipeline

## Part 1: Create the Classic Build Pipeline

1. Open **Azure DevOps**.

2. Open the **YouTube Clone** project.

3. Go to:

```text
Pipelines
```

4. Select:

```text
Create Pipeline
```

5. Select:

```text
Use the Classic Editor
```

6. Select the source:

```text
Azure Repos
```

7. Select the project:

```text
YouTube Clone
```

8. Select the repository:

```text
YouTube Clone
```

9. Select the required branch.

10. Select:

```text
Continue
```

11. Instead of using a predefined template, select:

```text
Empty Job
```

---

## Part 2: Configure the Pipeline

1. Set the pipeline name:

```text
YouTube Clone CI
```

2. Select the agent pool:

```text
Azure Pipelines
```

3. Use the Microsoft-hosted agent.

4. Select the agent specification:

```text
Ubuntu
```

5. Verify the **Get Sources** configuration.

6. Confirm that the Azure Repos repository and branch are selected.

7. The source code will be checked out automatically when the pipeline runs.

---

## Part 3: Add the npm Install Task

1. Under **Agent Job 1**, select:

```text
+
```

2. Search for:

```text
npm
```

3. Add the **npm** task.

4. Configure:

```text
Display Name: npm install
Command: install
```

5. Keep the **Working Folder** blank because `package.json` is located in the root directory.

6. Save the task configuration.

<img width="767" height="1241" alt="image" src="https://github.com/user-attachments/assets/492a1ebc-d98c-4eff-b836-0aa888e7fe80" />

The task will execute:

```bash
npm install
```

---

## Part 4: Add the npm Build Task

1. Add another **npm** task.

2. Select:

```text
Custom
```

3. Set the custom command to:

```text
run build
```

4. Set the display name:

```text
npm build
```

The resulting operation executes the equivalent of:

```bash
npm run build
```

==> **Additional Information:** The (Demo step stated) initially describes this as an npm custom command using `run build`. The YAML version later demonstrates the same requirement.

---

## Part 5: Publish the Build Artifact

1. Add a task.

2. Search for:

```text
Publish Build Artifact
```

3. Select **Publish Build Artifact**.

4. Configure the **Path to Publish** as:

```text
build
```

==> **Correction:** The initial pipeline configuration used an incorrect artifact path and caused `Not Found: Path to Publish`. Because the build folder is generated in the repository root, the path should be `build`.

5. Set the artifact name:

```text
drop
```

6. Keep the artifact publish location:

```text
Azure Pipelines
```

---

## Part 6: Add Azure App Service Deployment

1. Add another task.

2. Search for:

```text
Azure App Service Deploy
```

3. Select the task.

4. Set:

```text
Display Name: App Service Deploy
Connection Type: Resource Manager
```

5. Select the Azure **Service Connection**.

6. If no Service Connection exists, select **Authorize**.

7. Sign in using the required Microsoft account.

8. Verify that the Azure Service Connection is available in the dropdown.

==> **Additional Information:** The (Demo step stated) explains that the Service Connection provides the authenticated connection between Azure DevOps and Azure. It can use a Service Principal with appropriate Azure permissions.

9. Select:

```text
App Service Type: Web App on Linux
```

10. Select the Web App created previously.

11. Configure **Package or Folder** using the build directory:

```text
build
```
<img width="1364" height="644" alt="image" src="https://github.com/user-attachments/assets/1b7a97bf-d7dc-4b34-bb8a-41321b57b7fe" />

12. Configure the required Runtime Stack.

==> **Correction:** The (Demo step stated) initially selected `Node 18`, but later corrected the deployment configuration to `Static Site` for this application. The demonstrated working configuration uses **Static Site**.

---

## Part 7: Configure Pipeline Variables

1. Open the **Variables** section.

2. Review the system-defined variables.

3. For pipeline troubleshooting, locate:

```text
System.Debug
```

4. Change it from:

```text
false
```

to:

```text
true
```

5. Save the pipeline.

==> **Additional Information:** `System.Debug=true` enables additional debugging information and can be useful when troubleshooting pipeline failures.

---

## Part 8: Configure CI Trigger

1. Open the **Triggers** section.

2. Enable:

```text
Continuous Integration
```

3. Configure a schedule if scheduled execution is required.

4. Configure **Build Completion** if the pipeline needs to trigger after another pipeline completes.

---

## Part 9: Configure Pipeline Options

1. Open **Options**.

2. Review options such as:

- Work item linking.
- Create work item on failure.
- Build Job Timeout.
- Build Job Cancel Timeout.

3. The (Demo step stated) uses a Build Job Timeout example of:

```text
60 minutes
```

4. Review the pipeline history before execution.

---

## Part 10: Save and Run the Classic Pipeline

1. Select:

```text
Save and Queue
```

2. Add a comment:

```text
Initial Trigger
```

3. Review runtime options such as:

- Agent Pool.
- Agent Specification.
- Branch.
- Variables.

4. Select:

```text
Save and Run
```

5. Open the pipeline run.

6. Monitor the generated logs.

7. Verify the following sequence:

```text
Initialize Job
→ Checkout
→ npm install
→ npm build
→ Publish Build Artifact
→ Azure App Service Deploy
```

---

## Part 11: Troubleshoot the Artifact Publishing Failure

1. If **Publish Build Artifact** shows a red cross, open the failed task.

2. Check the error message.

The (Demo step stated) reports:

```text
Not Found: Path to Publish
```

3. Edit the pipeline.

4. Open:

```text
Publish Build Artifact
```

5. Change **Path to Publish** to:

```text
build
```

6. Save the pipeline.

7. Select:

```text
Save and Queue
```

8. Select:

```text
Save and Run
```

9. Open the pipeline logs again.

10. Verify that the artifact is successfully uploaded.

The (Demo step stated) reports that the build artifact was uploaded successfully after correcting the path.

---

## Part 12: Configure App Service Application Settings

1. Open the Azure Portal.

2. Open the Web App.

3. Go to:

```text
Configuration
```

4. Open **Application settings**.

5. Select:

```text
New application setting
```

6. Add:

```text
Name: WEBSITE_DYNAMIC_CACHE
Value: 0
```

7. Add another application setting:

```text
Name: CACHE_OPTION
Value: never
```

8. Save the configuration.

==> **Additional Information:** The (Demo step stated) uses these settings while troubleshooting the application continuing to display the Azure default page after deployment.

---

## Part 13: Restart the Web App

1. Open the Web App **Overview**.

2. Select:

```text
Stop
```

3. After the application stops, select:

```text
Start
```

4. Open the application URL again.

5. If the expected application is still not displayed, rerun the CI pipeline.

---

## Part 14: Correct the Runtime Stack

1. Return to the Azure DevOps pipeline.

2. Select:

```text
Edit Pipeline
```

3. Open the **Azure App Service Deploy** task.

4. Locate:

```text
Runtime Stack
```

5. Change the Runtime Stack from:

```text
Node 18
```

to:

```text
Static Site
```

6. Save the pipeline.

7. Select:

```text
Save and Run
```

8. Wait for the pipeline to complete.

9. Open the App Service URL.

10. Verify that the **YouTube Clone application** is now displayed.

11. Verify that videos are populated and the application is functioning.

==> **Correction:** The (Demo step stated) explicitly identifies the previous `Node 18` Runtime Stack selection as a mistake and changes it to `Static Site`.

---

# MAIN DEMO: YAML-Based Azure DevOps CI/CD Pipeline

## Part 1: Create a YAML Pipeline

1. Return to Azure DevOps.

2. Open:

```text
Pipelines
```

3. Select:

```text
New Pipeline
```

4. Select:

```text
Azure Repos Git
```

5. Select the **YouTube Clone** repository.

6. Select:

```text
Starter Pipeline
```

7. Azure DevOps generates the initial YAML structure.

---

## Part 2: Create the YAML Pipeline Structure

The demonstrated structure contains:

```text
Trigger
  ↓
Stages
  ↓
Jobs
  ↓
Pool / Agent
  ↓
Steps
  ↓
Tasks
```

The practical pipeline contains:

```text
Build Stage
    ↓
Build Job
    ↓
npm install
    ↓
npm run build
    ↓
Publish Artifact

Deploy Stage
    ↓
Deploy Job
    ↓
Download Artifact
    ↓
Azure App Service Deployment
```

==> **Additional Information:** The (Demo step stated) explains that a stage can contain multiple jobs, jobs run on agents, and jobs contain steps. Steps can be scripts or tasks.

---

## Part 3: Configure the Trigger

1. Add the pipeline trigger.

2. Configure the trigger for the `main` branch.

3. The trigger enables Continuous Integration.

==> **Correction:** The (Demo step stated) initially entered the trigger incorrectly and received:

```text
Unexpected value 'main', line one, column ten.
```

The branch must be represented using the proper YAML structure rather than placing `main` directly after `trigger:`.

A valid structure is:

```yaml
trigger:
  - main
```

---

## Part 4: Configure the Build Stage

1. Add:

```yaml
stages:
```

2. Create the Build stage:

```yaml
- stage: Build
```

3. Add the Build job:

```yaml
jobs:
- job: Build
```

4. Configure the agent pool:

```yaml
pool:
  vmImage: ubuntu-latest
```

5. Add the steps section:

```yaml
steps:
```

---

## Part 5: Add npm Install

1. Use the Azure DevOps task assistant if required.

2. Search for:

```text
npm
```

3. Select the npm task.

4. Configure the command:

```text
install
```

5. Keep the working folder at the default because `package.json` is in the repository root.

6. Add the generated task under:

```yaml
steps:
```

---

## Part 6: Add npm Build

1. Add another npm task.

2. Select:

```text
Custom
```

3. Set:

```text
Custom Command: run build
```

4. Add the generated task under the Build steps.

==> **Correction:** The (Demo step stated) initially generated an invalid npm task configuration where `run build` was associated with an unsupported command value. Azure DevOps reported valid values including `custom`; therefore the task must use `custom` with `run build` as the custom command.

---

## Part 7: Publish the Build Artifact

1. Add the **Publish Build Artifact** task.

2. Configure:

```text
Path to Publish: build
Artifact Name: drop
```

3. Add the generated task to the Build stage.

4. The Build stage now:

```text
npm install
→ npm run build
→ Publish Build Artifact
```

---

## Part 8: Create the Deploy Stage

1. Add another stage:

```yaml
- stage: Deploy
```

2. Add a Deploy job:

```yaml
- job: Deploy
```

3. Configure the agent:

```yaml
pool:
  vmImage: ubuntu-latest
```

4. Add:

```yaml
steps:
```

---

## Part 9: Download the Build Artifact

1. Add the **Download Build Artifact** task.

2. Select:

```text
Current Build
```

3. Select:

```text
Specific Artifact
```

4. Set the artifact name:

```text
drop
```

5. Configure the download destination.

6. Use the artifact directory or the default working directory.

==> **Additional Information:** The (Demo step stated) specifically demonstrates that artifacts published in one stage are not automatically available to another stage. Therefore, the Deploy stage must download the artifact before deployment.

---

## Part 10: Configure Azure App Service Deployment

1. Add the Azure App Service deployment task.

2. Search for:

```text
Web App
```

or:

```text
Azure App Service
```

3. Select the deployment task.

4. Select the Azure Service Connection.

5. Select:

```text
Web App on Linux
```

6. Select the App Service created earlier.

7. Configure the package/folder as the generated build directory.

The (Demo step stated) initially uses:

```text
build
```

and later corrects the package path to:

```text
System Default Working Directory
```

followed by:

```text
build
```

8. Configure the Runtime Stack as:

```text
Static
```

==> **Correction:** The (Demo step stated) explicitly corrects the package path to **System Default Working Directory + build** and uses the Static runtime configuration for the demonstrated application.

---

## Part 11: Validate the YAML Pipeline

1. Save the YAML pipeline.

2. Open:

```text
Validate
```

3. Review any YAML validation errors.

4. If the trigger produces:

```text
Unexpected value 'main'
```

use:

```yaml
trigger:
  - main
```

5. Validate the pipeline again.

6. Once validation succeeds, select:

```text
Save and Run
```

---

## Part 12: Troubleshoot the YAML npm Task

1. Open the failed pipeline run if the Build stage fails.

2. Open the npm task logs.

3. If the error states:

```text
unknown command run build
```

inspect the npm task configuration.

4. Open **Edit Pipeline**.

5. Locate the npm build task.

6. Change the command type to:

```text
custom
```

7. Set the Custom Command to:

```text
run build
```

8. Validate the YAML again.

9. Save the pipeline.

---

## Part 13: Run and Verify the Complete YAML Pipeline

1. Run the pipeline.

2. Verify the **Build** stage.

3. Confirm that:

```text
npm install
```

completes successfully.

4. Confirm that:

```text
npm run build
```

completes successfully.

5. Confirm that the build artifact is published.

6. Open the **Deploy** stage.

7. Confirm that the `drop` artifact is downloaded.

8. Confirm that the Azure App Service deployment completes successfully.

9. Verify that both stages are green.

10. Open the published artifact from the pipeline run.

11. Verify that the artifact is available.

12. Open the Azure App Service URL.

13. Verify that the **YouTube Clone application** is running.

The (Demo step stated)'s final verification confirms that the Build stage published an artifact, the Deploy stage downloaded it, and the application URL was working successfully.

---

# Practical Troubleshooting Summary

## Issue 1: Azure Repos Push Goes to GitHub

**Symptom:** `git remote -v` shows the GitHub repository as `origin`.

**Fix:**

```bash
git remote remove origin
git remote add origin <Azure-Repos-URL>
git push -u origin <branch-name>
```

---

## Issue 2: Publish Build Artifact Fails

**Error:**

```text
Not Found: Path to Publish
```

**Fix:**

```text
Path to Publish: build
```

The (Demo step stated) demonstrates that changing the path to `build` resolves the artifact publishing failure.

---

## Issue 3: Azure Default Web App Page Appears

**Action:**

1. Open Azure Portal.
2. Open the Web App.
3. Go to **Configuration**.
4. Add:

```text
WEBSITE_DYNAMIC_CACHE = 0
CACHE_OPTION = never
```

5. Save the configuration.
6. Stop the Web App.
7. Start the Web App.
8. Rerun the pipeline if necessary.

---

## Issue 4: Application Still Does Not Deploy Correctly

**Fix:**

Change the Azure App Service deployment Runtime Stack from:

```text
Node 18
```

to:

```text
Static Site
```

Then:

```text
Save and Run
```

---

## Issue 5: YAML Trigger Validation Error

**Error:**

```text
Unexpected value 'main'
```

**Fix:**

```yaml
trigger:
  - main
```

---

## Issue 6: YAML npm Build Failure

**Error:**

```text
unknown command run build
```

**Fix:**

Set the npm task command type to:

```text
custom
```

Then set:

```text
Custom Command: run build
```

---

## Issue 7: Artifact Is Not Available in Deploy Stage

**Cause:** The artifact was published in the Build stage but was not downloaded in the Deploy stage.

**Fix:** Add **Download Build Artifact** to the Deploy stage.

Configure:

```text
Download type: Single Artifact
Artifact name: drop
```

Then deploy from the downloaded artifact directory.

---

# Resource Cleanup

1. After completing the practical demonstration, return to the Azure Portal.

2. Identify the Resource Group created for the demonstration.

3. Delete the Resource Group when the resources are no longer required.

4. Verify that the associated resources have been removed.

==> **Additional Information:** The (Demo step stated) explicitly recommends cleaning up the Resource Group after the lab to avoid unnecessary Azure costs.

---

# Final Practical Flow

```text
GitHub YouTube Clone
        ↓
git clone
        ↓
Azure Repos
        ↓
Azure App Service
        ↓
Classic CI/CD Pipeline
        ↓
npm install
        ↓
npm run build
        ↓
Publish Artifact
        ↓
Azure App Service Deploy
        ↓
Application Verification
        ↓
YAML CI/CD Pipeline
        ↓
Build Stage
        ↓
Publish Artifact
        ↓
Deploy Stage
        ↓
Download Artifact
        ↓
Azure App Service
        ↓
Application Verification
```

## Scope of This Lab

The (Demo step stated) ends after successfully demonstrating the Build and YAML deployment flow. **Release Pipelines, Deployment Slots, Blue-Green Deployment, Deployment Quality Gates, and related topics are stated as topics for the next video**, so they are intentionally not included as completed labs here.





## 🚀 Next Step ( Depend on the This Current Demo [  Azure-DevOps-Build-Deploy-YouTube-Clone-hands-on-labs  ]- So Do immediatly after this one below steps 

👉 **[Continue to Demo 2 →](https://github.com/VyankateshwarTaikar/Azure-DevOps-Release-Pipeline-and-Blue-Green-Deployment-Hands-On-Lab-Guide.git)**
