# Lab. #5 - Automating Deployment

In this step, you will build a development treadmill, with the **OCI DevOps** service, which will automate the delivery of a containerized application to a Kubernetes cluster!

- 🌀 [Official OCI DevOps page](https://www.oracle.com/br/devops/devops-service/)
- 🧾 [OCI DevOps documentation](https://docs.oracle.com/pt-br/iaas/Content/devops/using/home.htm)

**You'll learn the entire step-by-step of this implementation.
 - [Pre Reqs: Collecting information relevant to the process](#PreReqs)
 - [Step 1: Clone repository and move content to DevOps project repository](#Passo1)
 - [Step 2: Create and configure Build process (CI)](#Passo2)
 - [Step 3: Create and configure artifact delivery (CI)](#Passo3)
 - [Step 4: Create and configure application delivery to kubernetes cluster (CD)](#Passo4)
 - [Step 5: Configure flow trigger and connect CI/CD pipelines](#Passo5)
 - [Step 6: Execution and testing](#Passo6)

 - - -

 ## <a name="PreReqs"></a> Pre Reqs: Collecting information relevant to the process

 1. [Log in](https://www.oracle.com/cloud/sign-in.html) to your OCI account

 2. Run the labs [Lab. #1](../Lab.%20%231%20-%20Resource%20Provisioning) and [Lab #2](../Lab.%20%232%20-%20Developing%20Cloud%20Native%20Applications%20-%20Parte%201).

 3. Go to the 🍔 hamburger menu: **Observability & Management** → **Application Performance** → **Administration**.

 ![](./Images/005-LAB4.png)

 4. In the bottom left-hand corner, under **Scope**,validate **Comparment** created in [Lab. #1](../Lab.%20%231%20-%20Resource%20Provisioning) is selected.

 5. Select the APM domain listed.
   
 ![](./Images/007-LAB4.png)

 6. Copy the domain's private key into a notebook.

 ![](./Images/008_1-LAB4.PNG)
 
 That's it! We've fulfilled all the prerequisites for the lab!
 
 - - -

 ## <a name="Passo1"></a> Step 1: Clone repository and move content to DevOps project repository

 1. Access the **Cloud Shell** by clicking on the icon as in the image below
 
 ![](./Images/013-LAB4.png)


 2. Clone the project repository.

 ```shell
 git clone https://github.com/CeInnovationTeam/BackendFTDev.git
 ```

 3. In the 🍔 hamburger menu, go to: **Developer Services** → **DevOps** → **Projects**.
  
 ![](./Images/014-LAB4.png)

 4. Access the listed project (created when provisioning the Resource Manager 😄).
  
 ![](./Images/015-LAB4.png)

 5. On the project page, click on **Create repository**.

 ![](./Images/016-LAB4.png)

 6. Fill in the form as follows:

   - **Name:** ftRepo
   - **Description:** (Define any description).
   - **Default branch:** main

 ![](./Images/017-LAB4.png)

 7. On the newly created repository page, click on **HTTPS** and:

- [1] Copy into notepad the information about the user to be used to work with git (**Git User**).
- 2] Copy the git clone command and run it in the Cloud Shell.

 ![](./Images/018-LAB4.png)

 8. In the Cloud Shell, when running the command, enter the newly copied **Git User** and your **Auth Token** as the password.

 9. At this point, the Cloud Shell should have two new directories:
 - BackendFTDev
 - ftRepo
 
 ![](./Images/019-LAB4.png)

 10. Run the following commands to copy the contents of the **BackendFTDev** repository to the **ftRepo** repository.

 ```shell
 git config --global user.email "<your-email>"
 git config --global user.name "<your-username>"
 cp -r BackendFTDev/* ftRepo/
 cd ftRepo
 git add -A
 git commit -m "Start of project"
 git push origin main
 ```

*At the end of the last command, the **Git user** and password (**Auth Token**) may be requested again*.

 ## <a name="Passo2"></a> Step 2: Create and configure Build process (CI)

1. Return to the DevOps project home page.
2. Click on **Create build pipeline**. 

 ![](./Images/020-LAB4.png)

 3. Fill in the form as follows and click on **Create**:
   - **Name**: build
   - **Description**: (Description).

 ![](./Images/021-LAB4.png)

 4. Open the newly created build pipeline.
 5. In the parameters tab, set the following parameters:
  - APM_ENDPOINT: *Information collected in the pre-requisites*.
  - APM_PVDATAKEY: *Information collected in the pre-requisites*.
  - APM_AGENT_URL: [🔗 copie este link](https://objectstorage.us-sanjose-1.oraclecloud.com/n/axfvmuge5kep/b/workshop_bucket/o/apm-java-agent-installer-1.11.1.4.jar)

  **WARNING** - When entering the name, value and description, click on the "+" sign so that the information is saved.
  
 ![](./Images/022-LAB4.png)

 6. Go to the **Build Pipeline** tab and click **Add Stage**.

 ![](./Images/023-LAB4.png)

 7. Select the **Managed Build** option and click **Next**.

 ![](./Images/024-LAB4.png)

 8. Fill in the form as follows:

- **Stage Name**: Creating artifacts
- **Description**: (Define any description).
- **OCI build agent compute shape**: *Do not change*.
- **Base container image**: *Do not change*.
- **Build spec file path**: *Do not change*.
      
![](./Images/025-LAB4.png)


9. Under Primary code repository, click **Select**, select the options below and click **Save**.

- **Source Connection type**: OCI Code Repository
- **Repositório**: ftRepo
- **Select Branch**: *Do not change*
- **Build source name**: java_root
    
![](./Images/026-LAB4.png)

- Once this is done, click on **Add**.

![](./Images/025_1-LAB4.png)

🤔 At this point it is important to understand how the tool works 📝.
    
- The tool uses a document in YAML format to define the steps that must be carried out during the process of building the application.
- By default, this document is called *build_spec.yaml* and must be configured in advance according to the application's needs.
- The steps will then be executed by a temporary instance (agent), which will be provisioned at the start of each execution and destroyed at the end of the process.
- 🧾 [Documentation on how to format the build document](https://docs.oracle.com/pt-br/iaas/Content/devops/using/build_specs.htm)
- 📑 [Document used in this workshop (build_spec.yaml)](https://raw.githubusercontent.com/CeInnovationTeam/BackendFTDev/main/build_spec.yaml)

 ## <a name="Passo3"></a> Step 3: Create and configure artifact delivery (CI)

 1. In the Build Pipeline tab, click on the **"+"** sign below the **Artifact Creation** stage and on **Add Stage**.
     
![](./Images/027-LAB4.png)


 2. Select the option **Deliver Artifacts** and click **Next**.
     
![](./Images/028-LAB4.png)


 3. Fill in the form as below and click on **Create artifact**.
 - **Stage name**: Artifact delivery
 - **Description**: (Define any description).

![](./Images/029_0-LAB4.png)

 4. In the artifact selection option, fill in as below and click **Add**.
- **Name**: backend_jar
 - **Type**: General artifact
 - Artifact registry**: *Select the Artifact registry generated by terraform with the name "artifact_repository"*.
- **Artifact location**: Set a Custom artifact location and version
- **Artifact path**: backend.jar
- **Version**: ${BUILDRUN_HASH}
- **Replace parameters used in this artifact**: Yes, substitute placeholders
       
![](./Images/030-LAB4.png)


5. Fill in the remaining field of the **Build config/result artifact name** table with "app" and click **Add**.
    
![](./Images/029_1-LAB4.png)

 6. In the Build Pipeline tab, click on the **"+"** sign below the **Artifact Delivery** stage and on **Add Stage**.

 ![](./Images/031-LAB4.png)

 7. Again, click on **Deliver Artifacts** and **Next**.

 ![](./Images/028-LAB4.png)

 8. Fill in the form as below and click on **Create Artifact**.
 - **Stage name**: Container Image Delivery
 - **Description**: (Define any description).

 ![](./Images/033_0-LAB4.png)
 
 9. In *Add artifact*, fill in the form as below and click **Add**.
- **Name**: backend_img
- **Type**: Container image repository
- **Artifact Source**: `<region code>.ocir.io/${IMG_PATH}`
- **Replace parameters used in this artifact**: Yes, substitute placeholders
   
*For Ashburn and São Paulo, the region codes are "iad" and "gru" respectively. If you are in another region, use the [reference table](https://docs.oracle.com/en-us/iaas/Content/General/Concepts/regions.htm)*.
       
![](./Images/032_0-LAB4.png)

9. Fill in the remaining field of the **Build config/result artifact name** table with: docker-img and click **Add**.
       
![](./Images/033_1-LAB4.png)

<a name="FinalPasso3"></a> This concludes the Build (CI) part of the project! So far we have automated the compilation of the java code, created the container image, and stored both in the artifact and container image repositories respectively. Now let's move on to Deployment (CD)!

## <a name="Passo4"></a> Step 4: Create and configure application delivery to Kubernetes cluster (CD)

1. In the **Cloud Shell**, to create the secret, run the commands below and enter your User OCID and Auth Token, collected previously.

 ```shell
  cd ftRepo/scripts/
  chmod +x create-secret.sh 
  ./create-secret.sh  
 ```

2. Wait for the end of the flow.
        
![](./Images/039-LAB4.png)

 3. Return to your DevOps project by clicking on the 🍔 hamburger menu and accessing: **Developer Services**  → **Projects**.
 4. In the left-hand corner, select **Environments**.
         
![](./Images/040-LAB4.png)

 5. Click on **Create New Environment**.

 6. Fill in the form below and click **Next**.
  - **Environment type**: Oracle Kubernetes Engine
  - **Name**: OKE
  - **Description**: OKE

 7. Select the Kubernetes Cluster, and click on **Create Envrinoment**.

 ![](./Images/041-LAB4.png)

 8. In the left-hand corner, select **Artifacts**, then **Add Artifact**.
          
![](./Images/042-LAB4.png)

 9. Fill in the form below and click **Add**.
 - **Name**: deployment.yaml
 - **Type**: Kubernetes manifest
 - **Artifact Source**: Inline
 - **Value**: Paste the contents of the file https://github.com/CeInnovationTeam/BackendFTDev/blob/main/scripts/deployment.yaml
 *Do not change the indentation (spaces) of the document, as this can break it*.
 - **Replace parameters used in this artifact**: Yes, substitute placeholders
          
![](./Images/043_0-LAB4.png)

 10. In the left-hand corner, select **Deployment Pipelines** and then click **Create Pipeline**.
          
![](./Images/044-LAB4.png)

 11. Fill in the form as below and click on **Create pipeline**.
 - **Pipeline name**: deploy
 - **Description**: (Define any description).
          
![](./Images/048-LAB4.png)

 12. In the **Parameters** Tab, set the following parameter:
 
 - REGISTRY_REGION: `<Region-Code>`.ocir.io  
          
![](./Images/049-LAB4.png)

 13. Return to the **Pipeline** tab and click on **Add Stage**.
          
![](./Images/050-LAB4.png)

 14. Select the option **Apply Manifest to your Kubernetes Cluster** and click **Next**.
          
![](./Images/051-LAB4.png)

 15. Fill in the form as follows:
 - **Name**: Application Deployment
 - **Description**: (Define any description).
 - **Environment**: OKE

![](./Images/052_0-LAB4.png)

16. Click on **Select Artifact**, and select **deployment.yaml**.

![](./Images/052_1-LAB4.png)

17. Once this is done, click on **Add**.
 
 That concludes the Deployment (CD) part of our project! In the next step, we'll connect both pipelines and define a trigger to start the automated process!

 ## <a name="Passo5"></a> Step 5: Configure flow trigger and connect CI/CD pipelines

  1. Return to the project by clicking on the 🍔 hamburger menu and accessing: **Developer Services** → **Projects**.
  2. In the left-hand corner, select **Triggers**, then click on **Create Trigger**.

  ![](./Images/053-LAB4.png)

  3. Fill in the form as below and click on **Create**.
  - **Name**: Home
  - **Description**: (Define any description).
  - **Source connection**: OCI Code Repository
  - **Select code repository**: ftRepo
  - **Actions**: Add Action
    - **Select Build Pipeline**: build
    - **Event**: Push (check) 
    - **Source branch**: main

![](./Images/054-LAB4.png)

*From this moment on, any new push made to the project repository will start the build pipeline created in this workshop*.

4. Return to the project's build pipeline configuration by selecting **Build Pipelines** → **build**.

![](./Images/055-LAB4.png)

  5. In the Build Pipeline tab, click on the **"+"** sign below the **Container Image Delivery** stage and click **Add Stage**.

![](./Images/056-LAB4.png)

6. Select the **Trigger Deployment** item and click **Next**.

![](./Images/057-LAB4.png)

7. Fill in the form as below and click **Add**.
- **Name**: Start of Deployment
- **Description**: (Define any description).
- **Select deployment pipeline**: deploy

*Keep the other fields unchanged*.

![](./Images/058-LAB4.png)

Congratulations on getting this far! Our pipeline is ready! In the next step we will validate the project, checking that everything is OK.

 ## <a name="Passo6"></a> Step 6: Execution and testing
  1.  Return to the project by clicking on the 🍔 hamburger menu and accessing: **Developer Services** → **Projects**.
  2.  Return to the project's build pipeline configuration by selecting **Build Pipelines** → **build**.
  
  ![](./Images/055-LAB4.png)

  3. In the top right-hand corner, select **Start Manual Run**.

![](./Images/055_1-LAB4.png)

  4. Keep the information from the standard form, and click on **Start Manual Run**.
  5. Wait for the flow to run.
  6. Access the Cloud Shell again and run the following command.

  ```shell
  kubectl get svc
  ```

  7. Copy the EXTERNAL-IP information from the _svc-java-app_ service as soon as it is available.

```shell
NAME           TYPE           CLUSTER-IP      EXTERNAL-IP       PORT(S)          AGE
kubernetes     ClusterIP      10.96.0.1       <none>            443/TCP          30h
svc-app        LoadBalancer   10.96.252.115   <svc-app-ip>   80:31159/TCP     29h
svc-java-app   LoadBalancer   10.96.16.229    <EXTERNAL-IP>   8081:32344/TCP   103m
```

  8. In the **Cloud Shell**, run the command below replacing the `<EXTERNAL-IP>` information with the copied IP.

   ```shell
  curl --location --request POST '<EXTERNAL-IP>:8081/processcart' \
--header 'Content-Type: application/json' \
--data '[
      {   "nome":"Oranges",
      "preco":1.99
      },
      {   "nome":"Apples",
          "preco":2.97
      },
      {   "nome":"Bananas",
          "preco":2.99
      },
      {   "nome":"Watermelon",
          "preco":3.99
      }
]'
```
- You should see the sum of the product prices! Try changing the values to check the sum!

![](./Images/059-LAB4.png)

### 👏🏻 Congratulations!!! You were able to successfully build a complete pipeline of **DevOps** in OCI! 🚀
