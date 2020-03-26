# REST API on top of IBM z/OS Mainframe CICS with OpenLegacy Adapter

The following demonstrate how to create an API for retrieval of customer credit cards by forwarding the request to an underlying CICS program using the OpenLegacy adapter.

## Pre-Requirements

- OpenLegacy IDE 4.5.2 (Full installation including JDK and all Maven dependencies).
- Internet Connection

## Demo Definition

- Creation of a new SDK Project.
- Generating Java Entities from **Cobol source** file
- Develop and run unit tests on the fly.
- Test the connectivity and data retrieval from the **Mainframe CICS Program**.
- Creation of an API Project on the top of your SDK Project.
  
![This PIC](createDockerProject.png)

## Step 1 – Create a New SDK Project

> First, we will create a new SDK project using the OpenLegacy IDE.
The purpose of the SDK project is to allow easy access to legacy backends, using standard and easy to use Java code.

1. Open the New Project Wizard:
   - File → New → OpenLegacy SDK Project
2. Define the **Project Name** as **mainframe-cics-native-sdk**
3. Click at the **Default Package** field, to automatically fill it up.
4. Select **Mainframe CICS TS** as the backend and click **Next**
5. Set the connection details to the backend based on following parameters:
    - **CICS Base URL:** `http://192.86.32.142`
    - **URI Map:** `prod/latest`
    - **CICS Port:** `12345`
    - **Code Page:** `CP037`
6. Click **Finish**

![How to create SDK Project](/docs/assets/mainframe/mainframe-cics-ol-adapter/createSDK.gif)

## Step 2 – Generate Java Model (Entity) from the Cobol Source

1. Copy the following resource to your samples folder at `src/main/resources/sample`:
    - [FININQ2.cbl](./resources/FININQ2.cbl)
2. **Right-Click** on the `FININQ2.cbl` file → OpenLegacy → Generate Model
3. **Execution Path**: `FININQ2`
4. Check **Generate JUnit Test checkbox**
5. Click **OK**

![How to parse Cobol](/docs/assets/mainframe/mainframe-cics-ol-adapter/parseCobol.gif)

## Step 3 – Create a JUnit Test

> OpenLegacy enables test-driven development by auto-generating test suites for each backend program (entities).
We can extend this test suite with additional unit tests to validate our connectivity to the backend.

1.  For this current JUnit test, we need only one usecse to check. 
Therefor, go to `src/test/java/tests/Fininq2Test.java` and mark the "fininq2TestUseCaseTest_2" method as comment.  

2. Go to `src/test/resources/mock/Fininq2Test`. 
There are four json files, two input json files and two output json files. 
Pay attention, the following two files are not in use(**usecase2 marked as comment**): 
test_fininq2Test_usecase_2.input
test_fininq2Test_usecase_2.output  

3. Copy the following json object in to the output json file "test_fininq2Test_usecase_1.output.json".

    
  ```
    {
      "dfhcommarea" : {
        "custId" : "412-83254",
        "creditCards" : [ {
        "cardNumber" : "4580123412341234",
        "cardType" : "GOLD",
        "cardLimit" : 5000,
        "cardUsage" : 1783
        }, {
        "cardNumber" : "4580002377826452",
        "cardType" : "PLATINUM",
        "cardLimit" : 10000,
        "cardUsage" : 567
        }, {
        "cardNumber" : "4580887386255265",
        "cardType" : "BUSINESS-G",
        "cardLimit" : 7000,
        "cardUsage" : 4873
        }, {
        "cardNumber" : "4580108372533424",
        "cardType" : "BASIC",
        "cardLimit" : 1000,
        "cardUsage" : 0
        }, {
        "cardNumber" : "4580773685986244",
        "cardType" : "FT-MEMBER",
        "cardLimit" : 2000,
        "cardUsage" : 600
        } ]
      }
    }
  ```

4. Edit the field 'custId' in the **input json file**(test_fininq2Test_usecase_1.input.json) with the value '412-83254' like the following:

 ```
"{
    "dfhcommarea" : {
      "custId" : "412-83254",
      "creditCards" : [ {
        "cardNumber" : "",
        "cardType" : "",
        "cardLimit" : 0,
        "cardUsage" : 0
        }, {
        "cardNumber" : "",
        "cardType" : "",
        "cardLimit" : 0,
        "cardUsage" : 0
        }, {
        "cardNumber" : "",
        "cardType" : "",
        "cardLimit" : 0,
        "cardUsage" : 0
        }, {
        "cardNumber" : "",
        "cardType" : "",
        "cardLimit" : 0,
        "cardUsage" : 0
        }, {
        "cardNumber" : "",
        "cardType" : "",
        "cardLimit" : 0,
        "cardUsage" : 0
        } ]
      }
    }
 ```
  
5. Go to `src/test/java/tests` 
6. Run the JUnit by Right Clicking on `Fininq2Test.java` → Run As → JUnit Tests (or press **F11**).
7. Check that the JUnit test runs without errors.

![This PIC](createDockerProject.JPG.png)

## Step 4 – Create APIs from SDK

1. Open the New Project Wizard:
   - File → New → OpenLegacy API Project
2. Define the **Project name** as `mainframe-cics-api`.
3. Click at the **Default Package** field, to automatically fill it up.
4. **This section should be implemented only in the case docker image creation is needed**  <br/>
	Checkbox microservice project and select docker at the deployment options.

![Create docker API](/docs/assets/mainframe/mainframe-cics-ol-adapter/createDockerProject.png)

5. Press Next and add the SDK project that was created in **Step 1**  as the reference project.
6. **Right-Click** on the **cards API** project → OpenLegacy → Generate API from SDK
    - Name the service `Cards`
    - Select from the `Fininq2` model the `custId` as input and change its name to `customerId` for better readability
    - Select from the `Fininq2` model the `creditCards` as output
    - **Click OK**
7. Uncheck **Use Input wrapper AND default method**
8. to `GET` andIn the API Editor, change the **HTTP Method** in the **Method** Tab  define the **Method path** as `/{customerId}` to align with REST best practices and save the changes

![Generate API](/docs/assets/mainframe/mainframe-cics-ol-adapter/generateAPI.gif)

## Step 5 - Run and Test your API

**The first section should be implemented only if it is docker API project**
1. Go to `src/main/java/com/mainframe_cics_ol_adapter_docker_api/openlegacy/config/MicroserviceConfiguration.java`.<br/>
** **Pay attention** ** <br/>
By default there is exclamation mark in the @Profile("!dev") annotation.
**Remove the exclamation mark (@Profile("dev")), if you want the application to run locally, otherwise leave it as is**.

![Runocker](/docs/assets/mainframe/mainframe-cics-ol-adapter/profileAnnotationInMicroservicesConfiguration.png)

2. **Right-Click** on the **cards API** project → OpenLegacy → Run Application
3. Open the browser on http://localhost:8080/swagger
4. Authorize through **Oauth2**
    - **Client Id:** `client_id`
    - **Client Secret:** `client_secret`
5. **Click** on the API we've created → Try it out

  - Set dummy customer ID as input (for example: `412-83254`)
  - You should see successful respond returned directly from the mainframe CICS program `FININQ2`!

    ```
    {
     "creditCards": [
       {
         "cardNumber": "4580123412341234",
         "cardType": "GOLD",
         "cardLimit": 5000,
         "cardUsage": 1783
       },
       {
         "cardNumber": "4580002377826452",
         "cardType": "PLATINUM",
         "cardLimit": 10000,
         "cardUsage": 567
       },
       {
         "cardNumber": "4580887386255265",
         "cardType": "BUSINESS-G",
         "cardLimit": 7000,
         "cardUsage": 4873
       },
       {
         "cardNumber": "4580108372533424",
         "cardType": "BASIC",
         "cardLimit": 1000,
         "cardUsage": 0
       },
       {
         "cardNumber": "4580773685986244",
         "cardType": "FT-MEMBER",
         "cardLimit": 2000,
         "cardUsage": 600
       }
     ]
    }
    ```  

![RUN API](/docs/assets/mainframe/mainframe-cics-ol-adapter/runAPI.gif)

# Summary

In this demo we have presented an end to end integration with Mainframe CICS using Openlegacy IDE within just a couple of minutes.
We have started from a COBOL source of a program we wanted to expose and automatically generated Java SDK that enables calling the underlying program, then we have presented the creation of a REST API utilizing the Mainframe CICS SDK.
We used the IDE to better model and design the API and showed how it works with a standard Swaager page.

