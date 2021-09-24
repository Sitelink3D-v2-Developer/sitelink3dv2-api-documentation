# Chapter 2 Sites
As its name suggests, many operations in the Sitelink3D v2 API require the context of a site. A site is simply the entity that configuration and work is associated with and is represented by an identifier string. 
Site managers will want to create sites for many reasons, including to:
-	Represent a physical job site visually a the map.
-	Organize data and resources associated with that site.
-	Monitor live work progress to ensure production is meeting expectations.
-	Run reports.

Sites are owned by Organizations (referred to in the API as Owners) and have properties including location, units and timezone. A single Owner may have zero or more Site associations. Both Sites and Owners are represented by identifier strings.
An owner is a representation of an account or logical business unit. The definition of an Owner is flexible. Owners may for example represent geographical regions, corporate subsidiaries, billing accounts or any other logical separation for which different authorization credentials may be desired. Owners are required for the creation and registration of Sites.
Data logging activity that occurs at a site is charged to the owning organization. This is implemented by deducting the appropriate number of service points from the organization’s account as discussed in Chapter 1.

## Microservice
```siteowner``` is the microservice that creates and manages sites. It is one of the first Sitelink3D v2 services that developers will acquaint themselves with. It is used to create and manage both sites and the cryptographic keys that secure those sites to guarantee their ownership. ```siteowner``` exposes operations that allow both online and offline site management. This chapter introduces some examples that demonstrate important role that ```siteowner``` plays in the Sitelink3D v2 micro service ecosystem. 

## Swagger
See https://api.sitelink.topcon.com/swagger/#/SiteOwner for detailed information on the ```siteowner``` endpoints.

## Site Discovery
Before looking at the examples it’s worth introducing an additional concept. One powerful feature of Sitelink3D v2 sites is Site Discovery. This is a technology that greatly improves the process by which clients can connect to and start working on a Sitelink3D v2 site. Site Discovery is being introduced here as it requires some additional configuration to enable.
Site Discovery is a new feature that allows clients to browse and connect to Sites in their vicinity using GPS enabled equipment such as a mobile phone. This replaces the previous process of connecting to specific sites using OpenVPN with a dedicated IP address and port. 

A Site Discovery enabled client application pushes notification to users when they enter what is known as a discoverable region (regions are defined in RDM as described in Chapter 3). This process allows client applications to ask users whether they wish to join sites. This approach eliminates any need for employees, contractors or site visitors to configure software or know anything about sites prior to arriving on location.

With Site Discovery, a client user only needs to be physically within a discoverable region and their client application will guide them through the process of site connection, presenting contact information where available. For example, a haul truck operator using the Topcon Haul App will be issued with a popup on their mobile device informing them of a site contact name and phone number as part of the discovery process. We will configure a site with this contact information in one of the following examples.

The power of Site Discovery becomes evident when considering cases where multiple client users are working at the same physical location. Each site manager can configure their own overlapping discoverable regions and the client user can select the site relevant to them using the options provided by the client application. 
There are three required configurations for site discovery:
-	A discoverable region must be defined in RDM. See Chapter 3. 
-	An Auth Code must be defined in RDM. See Chapter 3.
-	The ```discoverable``` flag must be enabled in the site’s RDM.

There is one optional configuration for site discovery:
-	Contact information can be associated with the site in RDM. See Chapter 3.

## Examples
The primary use of Site Owner in the example repository is for the creation and listing of sites. These operations can be seen in the ```create_site``` function found in ```components/sites/site_create/site_create.py``` and the ```list_sites``` function found in ```components/sites/site_list/site_list.py```.  Both of these operations require access to the owner identifier. Follow the steps in “Useful Tools” in Chapter 1 to determine the owner ID for your organization.

### Creating a Site
Let’s walk through the execution of the site creation example in the example repo. This process implements most of the functionality of the New Site Wizard in the Sitelink3D v2 web portal.

#### Configure the Example
Navigate to ```components/sites/site_create``` and open the wrapper script suitable for your platform; either ```site_create.bat``` or ```site_create.sh```.

Start populating the fields in the wrapper script, starting with the environment:
```
set env="qa"
set dc="us"
```

Populate the owner ID that you extracted from the browser console as described under “Useful Tools” in Chapter 1. Sitelink3D v2 uses this to determine which organization the site will be created under:

```
set owner_id="ae235e5e-6d87-4a84-80f2-0e56b137a131"
```

Populate the site name, position and timezone for the new site. The name will appear when the site is listed via the API or when viewed in any client application such as the Sitelink3D v2 web portal.

```set site_name="API Site Creation Demo 2"
set site_latitude="-27.979320763437187" 
set site_longitude="153.40316555667877"
set site_timezone="Australia/Brisbane"
```

The next configuration section relates to the contact details that are an optional property of Site Discovery. The ```create_site``` example utilizes these fields for illustration purposes but the configuration fields and the  ```contact``` frame can be removed from the ```post_bean_json``` static method in the ```SiteMetadataTraits``` class can be removed if not required in your application.

```set site_contact_name="Joe Burger"
set site_contact_email="jb@jb.com"
set site_contact_phone="123-456-7890
```

Lastly, this example requires JWT or OAuth credentials for authorization. See Chapter 1 for information on obtaining these tokens.

```
set jwt="eyJhbGciOiJSUzI1NiIsIn…6WyIqIl19LCJ -YBA-MSsdOxClylRNKE5taq7fjJ_lA"
rem # - or -
set oauth_id=""
set oauth_secret=""
set oauth_scope=""
```

#### Run the Script
Save the wrapper script and then run it in the console. The output will look something similar to the following.
```
> 2021-09-16 14:21:51,274 site_create INFO main:   Running site_create.py for server=https://us-qa-api.sitelink.topcon.com:443 dc=us owner=ce235e5e-6d87-4a84-80f2-0e56b137a132
> 2021-09-16 14:21:51,275 site_create INFO create_site:   post site creation to site owner https://us-qa-api.sitelink.topcon.com:443/siteowner/v1/owners/ce235e5e-6d87-4a84-80f2-0e56b137a132/create_site
> 2021-09-16 14:21:52,588 site_create INFO create_site:   post site definition to RDM https://us-qa-api.sitelink.topcon.com:443/rdm_log/v1/site/cd812dc65c699f924086eadbd886cc68c8ff7d459cd9cd32e51248696b211a39/domain/sitelink/events
> 2021-09-16 14:21:54,719 site_create INFO main:   Site cd812dc65c699f924086eadbd886cc68c8ff7d459cd9cd32e51248696b211a39 successfully created
```

The output tells us that site creation is actually a two-step process. First, we post the site creation frame to the ```siteowner``` micro service and then we post a definition of the site to RDM so that there is a record of the object that ```siteowner``` created in the system. Every object in Sitelink3D v2 must have an entry of an appropriate type representing it in RDM. Without a site entry in RDM, any site created by ```siteowner``` will not be visible in the system. See Chapter 3 for information on RDM.

#### Run the Script in Debug
Let’s increase the logging output in the example to ```DEBUG``` to understand what’s happening in more detail. See the Log Level section in Chapter 1 on logging configuration.

```
> 2021-09-16 14:29:19,213 connectionpool DEBUG _new_conn:   Starting new HTTPS connection (1): us-qa-api.sitelink.topcon.com:443
````

The script exchanges the OAuth credentials for a token using the ```oauth``` microservice.

```
> 2021-09-16 14:29:20,702 connectionpool DEBUG _make_request:   https://us-qa-api.sitelink.topcon.com:443 "POST /oauth/v1/token?grant_type=client_credentials&scope=9f2337a2400adf146ff7629c0a391079 HTTP/1.1" 200 372
```
The script prints a header line confirming the example running and its configuration.
```
> 2021-09-16 14:29:20,706 site_create INFO main:   Running site_create.py for server=https://us-qa-api.sitelink.topcon.com:443 dc=us owner=ce235e5e-6d87-4a84-80f2-0e56b137a132
```
The script posts a site creation frame to the ```siteowner``` microservice running in the QA environment. The owner ID is provided in the URL. The frame contains the following for the new site:
-	A uuid string uniquely identifying the site. Note that this is not the same as the site identifier that is returned by ```siteowner``` when the site is created.
-	A name to represent the site. Site names need not be unique.
-	The data center to create the site in. Best practice is to select the data center closest to the site. Once the site is created, the data center cannot be changed. 
-	An AsBuilt configuration determined by work type:
o	Small: Suitable for sites that mostly involve paving and curbing with a typical machine operating speed of 4 – 16 km/h.
o	Medium: Suitable for sites that involve bulk earth works, paving and compaction with a typical machine operating speed of 6 – 22 km/h. 
o	Large: Suitable for piste management with a typical machine operating speed of 10 – 42 km/h.

```
> 2021-09-16 14:29:20,707 site_create INFO create_site:   post site creation to site owner https://us-qa-api.sitelink.topcon.com:443/siteowner/v1/owners/ce235e5e-6d87-4a84-80f2-0e56b137a132/create_site
> 2021-09-16 14:29:20,708 site_create DEBUG create_site:   {
    "site_uuid": "e3f82b0c-f8b3-41c0-9fe8-f654c4b4662a",
    "name": "Example Site",
    "dc": "us",
    "region": "medium"
}
```

The ```siteowner``` microservice responds with a ```201``` and the following frame:
-	_id, rev, type are RDM concepts covered in Chapter 3.
-	Identifier is the string that will be used to reference this site in all future calls.

```
> 2021-09-16 14:29:21,047 connectionpool DEBUG _make_request:   https://us-qa-api.sitelink.topcon.com:443 "POST /siteowner/v1/owners/ce235e5e-6d87-4a84-80f2-0e56b137a132/create_site 
HTTP/1.1" 201 611
> 2021-09-16 14:29:21,048 site_create DEBUG create_site:   response from site owner {
    "_id": "e3f82b0c-f8b3-41c0-9fe8-f654c4b4662a",
    "rev": "a9b447d5-16a6-11ec-b606-0242ac11001e",
    "type": "site",
    "owner_uuid": "ce235e5e-6d87-4a84-80f2-0e56b137a132",
    "urn": "urn:X-topcon:le:b3668728-bde0-4f5f-8321-84299caad3a5:owner:ce235e5e-6d87-4a84-80f2-0e56b137a132:site:e3f82b0c-f8b3-41c0-9fe8-f654c4b4662a",
    "identifier": "6420a6ea32021e7460e4b1c86fcf4871aaa6d0ff7f3ac53d751fc363ee43e69c",
    "name": "Example Site",
    "dc": "us",
    "region": "medium",
    "owner_email": "owner-name@your-company.com",
    "archived": false,
    "create_timestamp_ms": 1631766560901,
    "modify_timestamp_ms": 1631766560901,
    "etag": "250ec8f4-3f21-4fb0-a599-c1894327feb2"
}
```

To make the site known within the whole Sitelink3D v2 ecosystems, we now must post a representation of this site to RDM via the ```rdm_log``` microservice. The site identifier returned by ```siteowner``` upon creation is used to create the ```rdm_log``` URL. Much of the metadata in the RDM frame will be recognizable fields from the New Site Wizard in the Sitelink3D v2 web portal. 

```
> 2021-09-16 14:29:22,055 site_create INFO create_site:   post site definition to RDM https://us-qa-api.sitelink.topcon.com:443/rdm_log/v1/site/6420a6ea32021e7460e4b1c86fcf4871aaa6d0ff7f3ac53d751fc363ee43e69c/domain/sitelink/events
> 2021-09-16 14:29:22,057 site_create DEBUG create_site:   {
    "_at": 1631766562055,
    "_id": "site",
    "_rev": "9ab433d9-1b5c-4d60-b939-55f526e05805",
    "_type": "sl::site",
    "_v": 3,
    "job_code": "code",
    "marker": {
        "lat": -27.979320763437187,
        "lon": 153.40316555667877
    },
    "contact": {
        "phone": "123-456-7890",
        "email": "jb@jb.com",
        "name": "Joe Burger"
    },
    "name": "Example Site",
    "timezone": "Australia/Brisbane"
}
```

RDM responds with a ```200``` confirming that the site has been recorded.

```
> 2021-09-16 14:29:24,100 connectionpool DEBUG _make_request:   https://us-qa-api.sitelink.topcon.com:443 "POST /rdm_log/v1/site/6420a6ea32021e7460e4b1c86fcf4871aaa6d0ff7f3ac53d751fc363ee43e69c/domain/sitelink/events HTTP/1.1" 200 63
> 2021-09-16 14:29:24,100 site_create DEBUG create_site:   response from RDM {
    "log_id": "a1801c35-e190-4eff-a18b-a0b0179b98e2",
    "seq": 1397496
}
```
Lastly the script confirms that the site was created successfully.
```
> 2021-09-16 14:29:24,116 site_create INFO main:   Site 6420a6ea32021e7460e4b1c86fcf4871aaa6d0ff7f3ac53d751fc363ee43e69c successfully created
```

#### The Code
Now that we have run the example, let’s take a look at the python code. The example scripts largely follow a similar pattern. A ```main``` function will perform the following steps:
-	Compile command line arguments.
-	Configure logging.
-	Create a ```ServerConfig``` object representing the session environment.
-	Create a header string based on the provided authorization.
-	Execute example specific code.

The principal function in this example is ```create_site``` which is called as follows:
```
site_id = create_site(a_site_name=args.site_name, a_dc=args.dc, a_server_config=server, a_owner_id=args.owner_id, a_latitude=args.site_latitude, a_longitude=args.site_longitude, a_phone=args.site_contact_phone, a_email=args.site_contact_email, a_name=args.site_contact_name, a_timezone=args.site_timezone, a_headers=headers)
```

As discussed above, Sites are identified by their identifier and the ```create_site``` function returns this identifier to calling code which caches it for subsequent API calls.

In this instance, the ```create_site``` function takes care of both the post to ```siteowner``` and to ```rdm_log``` so no further use of site identifier is required.

In contrast, the file ```create_site_configured_for_hauling.py``` demonstrates the creation of a site and a number of subsequent configurations at that site. One such configuration is the creation of a material definition which uses the site identifier returned by ```create_site```.
```
material_waste_id = create_material(a_site_id=site_id, a_server_config=server, a_material_name="Waste", a_headers=headers)
```
Incidentally, materials are one of many objects defined and managed by RDM, described in the next chapter.

### Listing Sites
The second example we’ll walk through is one that lists the sites currently owned by your organization. Running this example is a good way to inspect the results of the previous site creation example.
This example is called ```site_list``` found under ```components/sites/site_list```.

#### Configure the Example
Navigate to ```components/sites/site_list``` and open the wrapper script suitable for your platform; either ```site_list.bat``` or ```site_ list.sh```.
Start populating the fields in the wrapper script, starting with the environment and data center. We will retain the same settings as we used in the site creation example:

```
set env="qa"
set dc="us"
```

Populate the owner ID with the same value used to create the site:
```
set owner_id="ae235e5e-6d87-4a84-80f2-0e56b137a131"
```

Lastly, this example requires JWT or OAuth credentials for authorization. See Chapter 1 for information on obtaining these tokens. In the site creation example, we used a JWT. To illustrate the alternative, we will use OAuth to run the site list example. Note that either authorization method is valid for all examples.
```
set jwt=""
rem # - or -
set oauth_id="2dc51e38-8b90-479d-a45a-6be25284e465"
set oauth_secret="d481ab84-056c-454d-9615-f871a5f8558a"
set oauth_scope="9f2337a2400adf146ff7629c0a391079"
```

#### Run the Script
Save the wrapper script and then run it in the console. The output will look something similar to the following.

```
> 2021-09-22 11:24:55,286 site_list INFO main:   Running site_list.py for server=https://us-qa-api.sitelink.topcon.com:443 dc=us owner=ae235e5e-6d87-4a84-80f2-0e56b137a131
> 2021-09-22 11:24:55,287 site_list INFO list_sites:   get site list from site owner https://us-qa-api.sitelink.topcon.com:443/siteowner/v1/owners/ae235e5e-6d87-4a84-80f2-0e56b137a131/sites
> 2021-09-22 11:25:33,424 site_list INFO main:   Found 1 sites.
> 2021-09-22 11:25:33,425 site_list INFO main:   'Example Site' at lat:-27.979320763437187, lon:153.40316555667877 in timezone Australia/Brisbane
```

#### Run the Script in Debug
As with the site creation example, let’s increase the logging output in the example to ```DEBUG``` to understand what’s happening in more detail. See the Log Level section in Chapter 1 on logging configuration. 
The script prints a header line confirming the example running and its configuration.
```
> 2021-09-22 14:35:32,307 site_list INFO main:   Running site_list.py for server=https://us-api.sitelink.topcon.com:443 dc=us owner=ce235e5e-6d87-4a84-80f2-0e56b137a132
```
The script now performs a ```GET``` to the ```siteowner``` microservice at the ```sites``` endpoint to receive a list of the sites owned by the specified owner UUID.
```
> 2021-09-22 14:35:32,307 site_list INFO list_sites:   get site list from site owner https://us-api.sitelink.topcon.com:443/siteowner/v1/owners/ce235e5e-6d87-4a84-80f2-0e56b137a132/sites
> 2021-09-22 14:35:32,312 connectionpool DEBUG _new_conn:   Starting new HTTPS connection (1): us-api.sitelink.topcon.com:443
> 2021-09-22 14:35:34,478 connectionpool DEBUG _make_request:   https://us-api.sitelink.topcon.com:443 "GET /siteowner/v1/owners/ce235e5e-6d87-4a84-80f2-0e56b137a132/sites HTTP/1.1" 200 694
```

The ```siteowner``` microservice responds with a ```200``` and the following frame:
-	“next_key” is a pagination field that we will learn more about in Chapter 3. When this field is empty, the result set in unpaginated and complete.
-	“Items” is a list of the site frames returned by ```siteowner```
```
> 2021-09-22 14:35:34,478 site_list DEBUG list_sites:   response from site owner {
    "next_key": "",
    "items": [
        {
            "uuid": "e3f82b0c-f8b3-41c0-9fe8-f654c4b4662a",
            "identifier": "6420a6ea32021e7460e4b1c86fcf4871aaa6d0ff7f3ac53d751fc363ee43e69c",
            "name": "Example Site",
            "dc": "us",
            "region": "medium",
            "owner_email": "owner-name@your-company.com",
            "create_timestamp_ms": 1631766560901,
            "modify_timestamp_ms": 1632372400143,
            "archived": false,
            "owner_locked": false,
            "etag": "250ec8f4-3f21-4fb0-a599-c1894327feb2",
            "rdm_name": "Example Site",
            "rdm_code": "code",
            "rdm_marker": {
                "lat": -27.979320763437187,
                "lon": 153.40316555667877
            },
            "rdm_contact": {
               "email": "jb@jb.com",
            	"name": "Joe Burger",
            	"phone": "123-456-7890"
            },
            "rdm_timezone": "Australia/Brisbane"
        }    
    ]
}
```

Lastly the example prints a summary of the number of sites returned from the request and a summary line for each.
```
> 2021-09-22 14:35:34,487 site_list INFO main:   Found 1 sites.
> 2021-09-22 14:35:34,487 site_list INFO main:   'Example Site' at lat:-27.73729708959604, lon:153.25689945708956 in timezone Australia/Brisbane
```

#### The Code
The principal function in this example is ```list_sites``` which is called as follows:
sites = list_sites(a_server_config=server, a_owner_id=args.owner_id, a_headers=headers)
This is a simple function that issues a ```get``` to the ```siteowner``` microservice for the specified owner identifier. 
