# Chapter 1 Welcome Your Journey Starts Here

Welcome to the Topcon Sitelink3D v2 API example repository documentation. The chapters contained in this repository make reference to the Python scripts found in our Sitelink3D v2 Examples [here](https://github.com/Sitelink3D-v2-Developer/sitelink3dv2-examples). 

If you’re feeling daunted with where to start with our API and are looking for a guided walk through of the concepts, then you’re in the right place. We know what it’s like starting to learn a new system and its associated API. The swagger can be too detailed to start with and some of the high-level material sometimes seems more like marketing than a coding tool. That’s where this documentation fits in.

In writing this walkthrough, we are assuming a technical audience with construction domain knowledge but minimal experience with Sitelink3D v2. Concepts will be covered sequentially with sufficient detail to get you using the API.
So make a cup of coffee and let’s get started!
## Documentation Structure
These chapters introduce the following concepts:
### Sites
Most API operations rely on the context of a site. Whether you’ve already got a site created in the Sitelink3D v2 web portal or you’re relying entirely on the API, this chapter will introduce site basics and point to code examples that interact with ```siteowner```, the microservice responsible for site creation and management.
### RDM (Replicated Domain Model)
Don’t let the terminology put you off. RDM is simply the technology we’ve built for defining and distributing objects within the Sitelink3D v2 ecosystem. RDM is one of the simplest ways to get working with our API so we’ll be looking at examples of creating objects by writing to RDM and then using views to read data back.
### Files
Now that we know about sites and RDM, we have everything we need to start adding files to a Sitelink3D v2 site. File management is quite simple and is the foundation of importing design objects for use on machine control clients. In this chapter we’ll cover the basics and then move onto more advanced examples including file versioning and different domains.
### Design Data
We’re now ready to get your machines working with design data. In this chapter, we’ll be looking at how we import design data from an uploaded file and package it into a design set and task suitable for connected clients to download and display. If you don’t have a machine control client on hand, we’ll look at an example using our free Haul App to demonstrate how easy it is to get working with design data.
### Reports
Reporting is the first place to look for producing work output. In this chapter, we’ll introduce the fundamentals of Sitelink3D v2 reporting and the different types of reports available. This is an area where the API can be used to innovatively customize your visualizations. As a demonstration, we’ll introduce a GPX converter that allows our haul reports to be viewed in third party GPX viewers providing a map and speed / elevation plots.
### SmartView
Where reports produce historical information about site activity, SmartView is our live data streaming technology. SmartView is comprised of a suite of context specific apps running in the cloud. We call these SmartApps. This chapter introduces the basics of SmartView, how to connect to a SmartView SmartApp and how to stream and interpret the resulting data. SmartView streams are an excellent way to populate live site information on a dashboard or other radiator for your customers.
### AsBuilt
In this chapter, we introduce some concepts around AsBuilt using 3DMC as a machine control client for context. You’ll expand on your knowledge of Tasks from Chapter 5 and discover how sequences are used to group AsBuilt work. Lastly, we will demonstrate a point cloud report that provides AsBuilt output for your consumption.
## Example Code Environment
The examples in this repo are written and tested against Python 3.9. You can download Python from:
- [Windows](https://www.python.org/downloads/windows/)
- [Linux](https://www.python.org/downloads/source/)

Dependencies can be installed by running the python_setup.bat or python_setup.sh top level script in this repository as appropriate for your system. These scripts require PIP to be available in the system path.

for Windows users, run:
```
python_setup.bat
```
or, for Linux users, run:
```
python_setup.sh
```
## Repo Structure
The example code is split into two categories:
-	Components.
-	Workflows.
### Components
The Components folder contains examples of individual self-contained operations that can be performed using the API. These examples can be used to learn how basic semantic steps are implemented and what associated parameters and return values are involved. Component examples include:
-	Create a report.
-	Upload a file.
-	Add an operator.
Each component example is implemented in a function that can be included in other code. In this way, components can be grouped together to implement larger workflows.

### Workflows
The Workflows folder contains examples of how more elaborate functionality is implemented by aggregating various component examples together. These examples can be used to learn how the microservices and technologies work together in a broader context. Workflow examples include:
-	Convert a design file into a task able to be viewed on a machine control client.
-	Stream live information about the machines connected to a site using SmartView (chapter 7).
-	Configure a site with the metadata required to run the Topcon Haul App.
## Prerequisites
There are a few steps that need ticking off before we can start using the API. These prerequisites are listed below and covered in our Topcon Software Developer Network (TSDN) page at https://developer.topcon.com. 
### Create a Topcon Account
Creating a Topcon account is free. Follow the instructions at https://developer.topcon.com/en/gwWiCQ. Once you have an account, you can be invited to an Organization or create your own. 
### Create or Join an Organization
Creating an Organization is free. Follow the instructions at https://developer.topcon.com/en/gwWiCQ. Existing Organization owners can invite you to their organizations via email.
### Service Points
The Sitelink3D v2 API is charged under a consumption-based model. Consumption is reconciled by the deduction of preloaded service points. Service points are associated with an Organization and are consumed when data is logged to a site via our data-logger microservice on a per-client, per-day basis. Operations that do not write to data-logger will not consume service points. Connecting a client such as the Haul Truck app or 3DMC to a Sitelink3D v2 site will consume service points.
Although service points are required on both our QA and production environments, they are priced differently as described in the “Cloud Environments” section below. 
### Obtain API Credentials
Credentials are comprised of the following:
-	Client ID.
-	Client Secret.
-	Access Token.
-	Service Points for the organization (production only)

Obtaining these is explained at https://developer.topcon.com/en/igWiCQ. Note that step 5 of Method 2 is demonstrated in this example repository under components/tokens/get_token.py  
## Cloud Environments
Sitelink3D v2 is available at two isolated environments:
-	QA.
-	Production.

Our QA environment is a sand box intended for early testing of features before being deployed to production. 
Although service points are consumed on both environments, QA service points may be requested from Topcon free of charge.
Sitelink3D v2 is distributed over multiple data centers:
Production provides two data centers:
-	“US”.
-	“EU”.

QA only supports a “US” data center.
Some microservices are deployed only to the US datacenter. 
## Running the Examples
The examples in this repository are accompanied by Windows batch and Linux shell wrapper scripts. Although each example can be run directly from the python file with the appropriate command line arguments, these scripts allow easy configuration and execution of their associated python file. Perform the following steps to run an example:
-	Navigate to the desired example.
-	Configure the example using the wrapper script.
-	Run the script.

### Navigate to the Example
Browse the components and workflows folders in this repository as described above in the Repo Structure section. Select an example that best fits your needs.
### Configure the Example
For each example to run, the wrapper script appropriate to your platform must first be configured with certain settings. 
for Windows users, edit the `.bat` file and for Linux users, edit the `.sh` file to provide the required configuration. Each file may require different configuration depending on the purpose of the example but the following is typical:

The Sitelink3D v2 environment that the target Site resides on. The options are either "prod" or "qa". See section on “Cloud Environments” above.
```
env=""
```

The 64 alpha-numeric ID string of the target Site. This can be found in `the Sitelink3D v2 web portal -> Site menu -> Site Information` menu.
```
site_id=""
```

The Data Centre (dc) that the target Site was created on. The options are either “us” or “eu” on production or “us” only on QA.
```
dc=""
```

Your client OAuth details provided by the Sitelink3D v2 support team.
For more information, please refer to the `Sitelink3D v2 -> Integrating -> Getting Started` documentation on the [Topcon Software Developer Network](https://developer.topcon.com/en/) site.
```
oauth_id=""
oauth_secret=""
oauth_scope=""
```
## Authorization
The examples support two forms of authorization with sitelink3D v2:
-	OAuth
-	JWT

OAuth is the recommended authorization method and requires External User configuration in the Sitelink3D v2 web portal as referenced in the Configure the Example section above.
JWT (JSON Web Token) support is also provided in the example script configuration. Using a JWT may be convenient in some circumstances but comes at the cost of additional work to obtain and refresh the JWT for continued use. See the Useful Tools section below for information on how to obtain a JWT. 

## Example Output
### Log Level
The level of detail in the script output can be changed by altering the logging configuration in each python file. The default logging level is INFO. By setting to DEBUG, additional detail useful for development can be observed. The appropriate line will look like:
``` arg_parser = add_arguments_logging(arg_parser, logging.INFO)```
### Output Format
Each line of console output is formatted with the date, time, file, log level and function followed by the log message as follows:
```console
> 2021-09-16 14:21:51,274 site_create INFO main:  
```
The first output from each example is a line that confirms the example being run, the configured environment and any other useful settings relevant to the particular example as follows:
```console
2021-09-16 14:20:49,468 site_create INFO main:   Running site_create.py for server=https://us-qa-api.sitelink.topcon.com:443 dc=us owner=ce235e5e-6d87-4a84-80f2-0e56b137a132
```
## Useful Tools
At some points it can be necessary or useful to access Sitelink3D v2 web portal to obtain certain information. To access the Sitelink3D v2, follow the process in Step 4 at https://developer.topcon.com/en/gwWiCQ 

### Obtaining Owner Identifiers
Once logged in to the Sitelink3D v2 web portal, hit F12 in your Chrome browser to open a console and type in the following command ```SitelinkFrontend.core.store.getState().app.owner.ownerId``` to obtain the owner ID from the browser. 
### Obtaining JWTs
As discussed above in the Authorization section, the examples support both OAuth and JWT tokens. If both authorization types are provided, the examples will preference the JWT. It is important to understand that using OAuth credentials is more convenient in that a token exchange is made each time the examples are run. This makes the authorization “set-and-forget”. Using a JWT will require the JWT to be regularly updated in the wrapper scripts as these expire and are not refreshed automatically in the example scripts.
Once logged in to the Sitelink3D v2 web portal, hit F12 in your Chrome browser to open a console and type in the following command ``` SitelinkFrontend.core.store.getState().app.owner.jwt[0]``` to obtain the JWT currently being used by the browser. Note that this JWT will allow the scripts to perform any action that the current browser session is capable of.

## Your First Example
To put the above information into practice, in the next chapter we are going to walk through one specific example.

