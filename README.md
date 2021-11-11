# Your Documentation Journey Starts Here
Welcome to the Topcon Sitelink3D v2 API documentation repository. 

If you’re feeling daunted with where to start with our API and are looking for a guided walk-through of the concepts, then you’re in the right place. We know what it’s like starting to learn a new system and its associated API. The swagger can be too detailed to start with and some of the high-level material sometimes seems more like marketing than a coding tool. That’s where this documentation fits in.

In writing this walk-through, we are assuming a technical audience with construction domain knowledge but minimal experience with Sitelink3D v2. Concepts will be covered sequentially with sufficient detail to get you using the API.

Those interested in running the code examples referenced in this repository are invited to clone our ```sitelink3dv2-examples``` repository [here](https://github.com/Sitelink3D-v2-Developer/sitelink3dv2-examples).

So make a cup of coffee and let’s get started!

# Technical Overview
Sitelink3D v2 is a powerful and extensible cloud based construction site management tool suite. It is designed to provide productivity, visibility, and importantly seamless global interoperability between vendors, equipment and workers on construction sites. 

Sitelink3D v2 is designed as a collection of microservices. Each microservice performs a specific role such as enforcing security, managing sites or streaming data. Developers are free to interact with only those services required to implement their objectives. 

Although it is incredibly capable, Sitelink3D v2 at its heart facilitates a simple way of connecting machines to sites, sending work files to them and getting them working. At a high level, Sitelink3D v2 provides the following characteristics:

- High frequency telemetry to track the precise location and shape of machines on site.
- Collaborative design file management.
- Visualization of machine location, historic and real-time.
- Visualization of active design files, operators and tasks.
- AsBuilt surface recording and reporting for Intelligent Compaction and Bulk Earth Works.
- Site Discovery service to allow contractors to easily join a work site.
- Remote Support (Remote Control, Remote View) of operator HMI.
- Reporting services.

These characteristics are provided in an infrastructure that:

- Is scalable and fault tolerant.
- Allows OEMs to consume services with their own clients, applications and authentication.

# Repository Structure
The documentation in this repository is split into three categories:

- Setup.
- Services.
- Security.

## Setup
The Setup folder documents the initial configuration requirements for new users. This includes creating a Topcon account, acquiring service points and obtaining API credentials. This is the first step for new users.

## Services
The Services folder documents the functional blocks that comprise the Sitelink3D v2 microservice architecture. This includes sequential chapters that introduce the concepts of sites, design data, reports and more in a logical order. This is the second step once setup is completed.

## Security
The Security folder documents the Sitelink3D v2 security architecture. This includes a walk-through of token usage and how our microservices enforce resource access policies. Although not essential reading, it provides good background for curious developers.

# Getting Help
If you require any assistance on your Sitelink3D v2 API journey, please contact us [here](mailto:sitelink3d-api-support@topcon.com).
