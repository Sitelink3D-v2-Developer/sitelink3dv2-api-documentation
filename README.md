# Your Documentation Journey Starts Here
Welcome to the Topcon Sitelink3D v2 API documentation repository. 

If you’re feeling daunted with where to start with our API and are looking for a guided walk through of the concepts, then you’re in the right place. We know what it’s like starting to learn a new system and its associated API. The swagger can be too detailed to start with and some of the high-level material sometimes seems more like marketing than a coding tool. That’s where this documentation fits in.

In writing this walkthrough, we are assuming a technical audience with construction domain knowledge but minimal experience with Sitelink3D v2. Concepts will be covered sequentially with sufficient detail to get you using the API.

Those interested in running the code examples referenced in this repository are invited to clone our ```sitelink3dv2-examples``` repository [here](https://github.com/Sitelink3D-v2-Developer/sitelink3dv2-examples).

So make a cup of coffee and let’s get started!

## Repository Structure
The documentation in this repository is comprised of sequential chapters which introduce the following concepts:

### 1. Sites
Most API operations rely on the context of a site. Whether you’ve already got a site created in the Sitelink3D v2 web portal or you’re relying entirely on the API, this chapter will introduce site basics and point to code examples that interact with ```siteowner```, the microservice responsible for site creation and management.

### 2. RDM (Replicated Domain Model)
Don’t let the terminology put you off. RDM is simply the technology we’ve built for defining and distributing objects within the Sitelink3D v2 ecosystem. RDM is one of the simplest ways to get working with our API. In this chapter we’ll be looking at examples of creating objects by writing to the ```rdm``` microservice and then using views to read data back.

### 3. Files
Now that we know about sites and RDM, we have everything we need to start adding files to a Sitelink3D v2 site. File management is quite simple and is the foundation of importing design objects for use on machine control clients. In this chapter we’ll cover the basics of the ```file``` microservice and then move onto more advanced examples including file versioning and different domains.

### 4. Design Data
We’re now ready to get your machines working with design data. In this chapter, we’ll be looking at how we use the ```designfile``` microservice to import design data from an uploaded file. If you don’t have a machine control client on hand, we’ll look at an example using our free Haul App to demonstrate how easy it is to get working with design data.

### 5. Reports
Reporting is the first place to look for producing work output. In this chapter, we’ll introduce the fundamentals of the ```reporting``` microservice and the different types of reports available. This is an area where the API can be used to innovatively customize your visualizations. As a demonstration, we’ll introduce a GPX converter that allows our haul reports to be viewed in third party GPX viewers providing a map and speed / elevation plots.

### 6. SmartView
Where reports produce historical information about site activity, SmartView is our live data streaming technology. SmartView is comprised of a suite of context specific apps running in the cloud. We call these SmartApps. This chapter introduces the basics of the ```smart_view``` microservice, how to connect to a SmartView SmartApp and how to stream and interpret the resulting data. SmartView streams are an excellent way to populate live site information on a dashboard or other radiator for your customers.

### 7. AsBuilt
In this chapter, we introduce some concepts around AsBuilt using 3DMC as a machine control client for context. You’ll expand on your knowledge of Tasks from Chapter 4 and discover how sequences are used to group AsBuilt work. Lastly, we will demonstrate a point cloud report that provides AsBuilt output for your consumption.

# Getting Help
If you require any assistance on your Sitelink3D v2 API journey, please contact us [here](mailto:sitelink3d-api-support@topcon.com).
