# Chapter 3 Files
Sitelink3D v2 supports a general purpose file storage hierarchy. Files of any size and type can be uploaded, versioned, downloaded and nested under folders. As with other objects in Sitelink3D v2, files and folders have representations in RDM which defines the properties and structure of the file system. Files and folders are specific to a site.

# File Domains
There are two domains that files may exist in within Sitelink3D v2:

- File System.
- Operator.

## File System Domain
The file system domain contains the files and folders visible under the Site Files tab in the Sitelink3D v2 web portal File Manager. These files may contain any data including:

- Design data.
- Site SOPs.
- Localization.
- Instructional videos.

## Operator Domain
The operator domain contains topo files visible under the Operator Files tab in the Sitelink3D v2 web portal File Manager. These files contain topo data typically produced by machines in the field. Operator domain files are organized by the machine operator rather than a folder hierarchy. There is hence a set of operator files for each user that saves topo data in a machine running 3DMC and synchronizes that data to Sitelink3D v2. 

# Folders
Folders are very similar to files in that they are created by posting RDM representations.

# Examples
The example repository contains various file related code.

| Example                                       | Location         | Purpose                                                                                                            |
|-----------------------------------------------|------------------|--------------------------------------------------------------------------------------------------------------------|
| file_download                                 | components/files | Download a previously uploaded file.                                                                               |
| file_features                                 | components/files | Inspect the contents of a previously uploaded file for design data.                                                |
| file_list                                     | components/files | List the files and folders in the operator and file_system domains.                                                |
| file_upload                                   | components/files | Upload a file.                                                                                                     |
| download_operator_pt3_files_as_landxml        | workflows        | Convert a pt3 operator file in the operator domain to landxml and download both pt3 and xml files.                 |
| upload_and_query_files_with_multiple_versions | workflows/files  | Upload multiple versions of the same file to demonstrate versioning and list all files with more than one version. |


## Listing Files
Let’s walk through the execution of the file list example in the example repository.

### Configure the Example
Navigate to ```components/files/file_list``` and open the wrapper script suitable for your platform; either ```file_list.bat``` or ```file_list.sh```.

Start populating the fields in the wrapper script, starting with the environment:
```
set env="qa"
set dc="us"
```

Populate the site ID that you wish to list the files for. This value can be determined from the Sitelink3D v2 web portal or by executing the ```site_list``` example described in the site chapter of this documentation:

```
set site_id="fdc59e465e0476fbf5bebc1314bf73c23b7ee0c1a095716b57a7440e1db4cd33"
```

Lastly, this example requires JWT or OAuth credentials for authorization.

```
set jwt="eyJhbGciOiJSUzI1NiIsIn…6WyIqIl19LCJ -YBA-MSsdOxClylRNKE5taq7fjJ_lA"
rem # - or -
set oauth_id=""
set oauth_secret=""
set oauth_scope=""
```

### Run the Script
Save the wrapper script and then run it in the console. The output will look something similar to the following.

```
> 2021-11-17 10:33:01,738 file_list INFO main:   Running file_list.py for server=https://us-qa-api.sitelink.topcon.com:443 dc=us site=fdc59e465e0476fbf5bebc1314bf73c23b7ee0c1a095716b57a7440e1db4cd33
> 2021-11-17 10:33:04,100 file_list INFO main:   Found 191 files in the 'file_system' domain
> 2021-11-17 10:33:04,105 file_list INFO main:   Found 3 files in the 'operator' domain
```

### Run the Script in Debug
As with the site creation example, let’s increase the logging output in the example to ```DEBUG``` to understand what’s happening in more detail. The script prints a header line confirming the example running and its configuration.

```
> 2021-11-17 10:43:34,296 file_list INFO main:   Running file_list.py for server=https://us-qa-api.sitelink.topcon.com:443 dc=us 
site=fdc59e465e0476fbf5bebc1314bf73c23b7ee0c1a095716b57a7440e1db4cd33
```

The script now performs a ```GET``` to the ```RDM``` microservice at the ```file_system``` domain using the ```v_fs_files_by_folder``` view.

```
> 2021-11-17 10:43:35,931 connectionpool DEBUG _make_request:   https://us-qa-api.sitelink.topcon.com:443 "GET 
/rdm/v1/site/fdc59e465e0476fbf5bebc1314bf73c23b7ee0c1a095716b57a7440e1db4cd33/domain/file_system/view/v_fs_files_by_folder?limit=500 HTTP/1.1" 200 None
```

The ```RDM``` microservice responds with a ```200``` and the following (truncated) frame.

```
> 2021-11-17 10:43:36,545 file_list DEBUG query_files:   Files listing result for site fdc59e465e0476fbf5bebc1314bf73c23b7ee0c1a095716b57a7440e1db4cd33, domain file_system, view v_fs_files_by_folder 
[
    {
        "id": "705143fe-72ec-428a-bbb0-a7497aca1a4e",
        "key": [
            "03bf1a00-a61b-45b2-98d1-b31c44ff7e14",
            "fs::file",
            "tps-bris.tp3"
        ],
        "value": {
            "_at": 1624586682746,
            "_id": "705143fe-72ec-428a-bbb0-a7497aca1a4e",
            "_rev": "36b43174-4c23-4b52-bd81-9b5dc3a1f596",
            "_type": "fs::file",
            "_v": 0,
            "name": "tps-bris.tp3",
            "parent": "03bf1a00-a61b-45b2-98d1-b31c44ff7e14",
            "size": 137334,
            "uuid": "705143fe-72ec-428a-bbb0-a7497aca1a4e"
        }
    },
    {
        "id": "29319307-6421-40c3-95af-982218bae71f",
        "key": [
            "09b14694-42c3-4e17-919f-8dcf10b59778",
            "fs::file",
            "example_file.txt"
        ],
        "value": {
            "_at": 1634098967702,
            "_id": "29319307-6421-40c3-95af-982218bae71f",
            "_rev": "8d535100-bcfe-4888-bdc1-9456c9302a41",
            "_type": "fs::file",
            "_v": 0,
            "name": "example_file.txt",
            "parent": "09b14694-42c3-4e17-919f-8dcf10b59778",
            "size": 124,
            "uuid": "43db765d-f916-4232-85f5-018cfd8a57c3"
        }
    },
    ...
    {
        "id": "7ebe29ae-e32d-4949-8c6c-2c410b084770",
        "key": [
            null,
            "fs::folder",
            "web test"
        ],
        "value": {
            "_at": 1634093904767,
            "_id": "7ebe29ae-e32d-4949-8c6c-2c410b084770",
            "_rev": "a35a9106-137c-4691-b889-922ac9a43282",
            "_type": "fs::folder",
            "name": "web test"
        }
    }
]
```

The script then performs a ```GET``` to the ```RDM``` microservice at the ```operator``` domain using the ```v_fs_files_by_operator``` view. Note that the appropriate view for the requested domain must be used.

```
> 2021-11-17 10:43:37,218 connectionpool DEBUG _make_request:   https://us-qa-api.sitelink.topcon.com:443 "GET /rdm/v1/site/fdc59e465e0476fbf5bebc1314bf73c23b7ee0c1a095716b57a7440e1db4cd33/domain/operator/view/v_op_files_by_operator?limit=500 HTTP/1.1" 200 1271

```

Again the ```RDM``` microservice responds with a ```200``` and the following frame.

```
> 2021-11-17 10:43:37,218 file_list DEBUG query_files:   Files listing result for site fdc59e465e0476fbf5bebc1314bf73c23b7ee0c1a095716b57a7440e1db4cd33, domain operator, view v_op_files_by_operator 
[
    {
        "id": "84594f28-faf6-4aa2-9f1b-5f18d3eca21f",
        "key": [
            "42cfb4e7-656b-4646-93d4-6c56f699caf0",
            "New My Data.pt3"
        ],
        "value": {
            "_archived": false,
            "_at": 1633648648999,
            "_id": "84594f28-faf6-4aa2-9f1b-5f18d3eca21f",
            "_rev": "f7dc150e-a45f-4d33-a9ab-79d6ef84bc26",
            "_type": "op::file",
            "name": "New My Data.pt3",
            "operator": "42cfb4e7-656b-4646-93d4-6c56f699caf0",
            "sitelink_file_id": "7e9a250b-d44f-4fff-9070-ac29194f53cd",
            "size": 790
        }
    },
    {
        "id": "0751fffe-28f5-444a-a05a-961331c5586a",
        "key": [
            "c9022227-aff7-4d4e-b63e-d03d6aebe497",
            "My_Operator_Data.pt3"
        ],
        "value": {
            "_at": 1635997956986,
            "_id": "0751fffe-28f5-444a-a05a-961331c5586a",
            "_rev": "47d70e4e-54d7-4cf5-9905-889e16f82ac9",
            "_type": "op::file",
            "_v": 0,
            "name": "My_Operator_Data.pt3",
            "operator": "c9022227-aff7-4d4e-b63e-d03d6aebe497",
            "sitelink_file_id": "0751fffe-28f5-444a-a05a-961331c5586a",
            "size": 872
        }
    },
    {
        "id": "3a0c1ea8-02e1-4e44-9a51-e9e88f54be0c",
        "key": [
            "e5ba2644-1f46-4f9b-b52a-5d7529742561",
            "my data layer.pt3"
        ],
        "value": {
            "_archived": false,
            "_at": 1633930074304,
            "_id": "3a0c1ea8-02e1-4e44-9a51-e9e88f54be0c",
            "_rev": "4497fa59-c23f-4f42-add4-47ee869cb9e0",
            "_type": "op::file",
            "name": "my data layer.pt3",
            "operator": "e5ba2644-1f46-4f9b-b52a-5d7529742561",
            "sitelink_file_id": "b86111ac-f9b5-478a-8773-f0c6af5eca45",
            "size": 1036
        }
    }
]
```

Lastly the script prints the trailing summary for both domains.
```
> 2021-11-17 12:25:13,625 file_list INFO main:   Found 191 files in the 'file_system' domain
> 2021-11-17 12:25:13,625 file_list INFO main:   Found 3 files in the 'operator' domain
```

### The Code
The principal function in this example is ```query_files``` which is called once for each domain / view pair as follows:
```
filesystem_domain_list = query_files(a_server_config=server, a_site_id=args.site_id, a_page_limit=args.page_limit, a_start=args.start, a_domain="file_system", a_view="v_fs_files_by_folder", a_headers=headers)
operator_domain_list = query_files(a_server_config=server, a_site_id=args.site_id, a_page_limit=args.page_limit, a_start=args.start, a_domain="operator", a_view="v_op_files_by_operator", a_headers=headers)
```

This is a simple function that issues a ```get``` to the ```rdm``` microservice for the specified domain and view:
```
rdm_list_files_url = "{0}/rdm/v1/site/{1}/domain/{2}/view/{3}".format(a_server_config.to_url(), a_site_id, a_domain, a_view)
```

## Uploading Files
Let’s walk through the execution of the file upload example in the example repository.

### Configure the Example
Navigate to ```components/files/file_upload``` and open the wrapper script suitable for your platform. The examples are broken down into domains to demonstrate the different configuration. 

#### Operator Upload Specific Configuration

To upload a file to the operator domain open either ```file_operator_upload.bat``` or ```file_operator_upload.sh```. 

The operator domain contains topo data generated by connected machines. This script can be used to send topo data to machines by targeting the operator active in that machine. 
Files uploaded by this script are visible in the Sitelink3D v2 web site File Manager on the "Operator Files" tab. This script is identical to the ```file_upload.bat``` example aside from:

1. The target domain is configured as "operator" rather than "file_system" which Sitelink3D v2 interprets as operator topo data.
2. The sample file to upload is a PT3 point file, typical of one created on a machine by surveying topo points into a layer.
3. Additional comments are provided below that describe how the operator uuid to associate the uploaded file with is determined.

Specify the name of the operator file to upload. An example PT3 file is provided with the script to demonstrate the upload:
```
set file_name="My_Operator_Data.pt3"
```

A parent_uuid is _required_ when writing to the "operator" domain. When writing to the operator domain, this UUID determines the target operator folder on the "Operator Files" tab. To find the UUID for an existing operator, perform the following steps:

1. Navigate the example repository to the file ```components/metadata/metadata_list/metadata_list.py``` and ensure the logging level is set to ```logging.DEBUG```.
2. Populate the wrapper script suitable for your platform (```metadata_list.bat``` or ```metadata_list.sh```) with the details of your site and credentials.
3. Run the wrapper script to list all metadata at your site. Redirect the output to a file to make the contents easy to inspect.
4. Search the output file for the operators available at the site and identify the associated id field for the operator of interest as follows.
5. Copy the operator id into the "parent_uuid" field in the ```file_operator_upload.bat``` or ```file_operator_upload.sh``` file as demonstrated below.

```
> 2021-07-13 14:29:29,806 metadata_list INFO main:   Found Operator 'John Galt'.
> 2021-07-13 14:29:29,806 metadata_list DEBUG main:   {
     "id": "187d83e1-465b-48de-bed8-cef49e8d678a",
     "key": [
         "Galt",
         "John"
     ],
     "value": {
         "_at": 1625184640699,
         "_id": "187d83e1-465b-48de-bed8-cef49e8d678a",
         "_rev": "f0d091ae-6f8b-43c7-bef2-a8335b673f90",
         "_type": "sl::operator",
         "firstName": "John",
         "lastName": "Galt"
     }
}
```

The "id" field represents the operator ID to associate the file upload with. This is configured as the ```parent_uuid``` variable.

```
set parent_uuid="187d83e1-465b-48de-bed8-cef49e8d678a"
```

#### File System Upload Specific Configuration

To upload a file to the file_system domain open either ```file_upload.bat``` or ```file_upload.sh```. 

A parent_uuid is _not required_ when writing to the "file_system" domain. When writing to the file_system domain, this UUID determines the optional folder UUID to nest the file or folder under. To upload to the root of the file system simply leave the variable blank:
```
set parent_uuid=""
```

#### General Configuration

Start populating the fields in the wrapper script, starting with the environment:
```
set env="qa"
set dc="us"
```

Populate the site ID that you wish to list the files for. This value can be determined from the Sitelink3D v2 web portal or by executing the ```site_list``` example described in the site chapter of this documentation:

```
set site_id="fdc59e465e0476fbf5bebc1314bf73c23b7ee0c1a095716b57a7440e1db4cd33"
```

Lastly, this example requires JWT or OAuth credentials for authorization.

```
set jwt="eyJhbGciOiJSUzI1NiIsIn…6WyIqIl19LCJ -YBA-MSsdOxClylRNKE5taq7fjJ_lA"
rem # - or -
set oauth_id=""
set oauth_secret=""
set oauth_scope=""
```

### Run the Script
Save the wrapper script and then run it in the console. Note that the same python file is used to implement both the file_system and operator upload demonstrations as the distinction is only a function of the configuration which is handled by the wrapper scripts. If successful, the script simply prints a header line confirming the example running and its configuration:

```
> 2021-11-17 13:01:02,814 file_upload INFO main:   Running file_upload.py for server=https://us-qa-api.sitelink.topcon.com:443 dc=us site=fdc59e465e0476fbf5bebc1314bf73c23b7ee0c1a095716b57a7440e1db4cd33
```

### Run the Script in Debug
As with the site creation example, let’s increase the logging output in the example to ```DEBUG``` to understand what’s happening in more detail. The script prints a header line confirming the example running and its configuration.

#### Operator Upload Specific Configuration
```
> 2021-11-17 13:03:12,732 file_upload INFO main:   Running file_upload.py for server=https://us-qa-api.sitelink.topcon.com:443 dc=us site=fdc59e465e0476fbf5bebc1314bf73c23b7ee0c1a095716b57a7440e1db4cd33
```

The script then creates a file upload payload based on the configuration. Included is the provided file name, that file's size and a generated UUID to internally refer to the file within Sitelink3D v2.
```
> 2021-11-17 13:03:12,733 file_upload DEBUG upload_file:   Upload file to https://us-qa-api.sitelink.topcon.com:443/file/v1/sites/fdc59e465e0476fbf5bebc1314bf73c23b7ee0c1a095716b57a7440e1db4cd33/upload
> 2021-11-17 13:03:12,733 file_upload DEBUG upload_file:   File Upload payload: {
    "upload-uuid": "6d205ba1-71f4-44ad-871f-e57929750889",
    "upload-file-name": "My_Operator_Data.pt3",
    "upload-file-size": 872
}
> 2021-11-17 13:03:12,738 connectionpool DEBUG _new_conn:   Starting new HTTPS connection (1): us-qa-api.sitelink.topcon.com:443
> 2021-11-17 13:03:14,116 connectionpool DEBUG _make_request:   https://us-qa-api.sitelink.topcon.com:443 "POST /file/v1/sites/fdc59e465e0476fbf5bebc1314bf73c23b7ee0c1a095716b57a7440e1db4cd33/upload?upload-uuid=6d205ba1-71f4-44ad-871f-e57929750889&upload-file-name=My_Operator_Data.pt3&upload-file-size=872 HTTP/1.1" 200 37
```

To ensure the uploaded file is visible within Sitelink3D v2, an RDM representation is then posted. The script generates an ```op::file``` type RDM payload representing the file just uploaded and posts this to RDM:
```
> 2021-11-17 13:03:14,117 file_upload DEBUG upload_file:   File RDM payload: {
    "_id": "6d205ba1-71f4-44ad-871f-e57929750889",
    "name": "My_Operator_Data.pt3",
    "size": 872,
    "_rev": "fe53a9a4-691d-4408-bd12-34ba34e43bfd",
    "_v": 0,
    "_at": 1637118192732,
    "sitelink_file_id": "6d205ba1-71f4-44ad-871f-e57929750889",
    "_type": "op::file",
    "operator": "187d83e1-465b-48de-bed8-cef49e8d678a"
}
> 2021-11-17 13:03:14,118 file_upload DEBUG upload_file:   Upload RDM to https://us-qa-api.sitelink.topcon.com:443/rdm_log/v1/site/fdc59e465e0476fbf5bebc1314bf73c23b7ee0c1a095716b57a7440e1db4cd33/domain/operator/events
> 2021-11-17 13:03:14,429 connectionpool DEBUG _make_request:   https://us-qa-api.sitelink.topcon.com:443 "POST /rdm_log/v1/site/fdc59e465e0476fbf5bebc1314bf73c23b7ee0c1a095716b57a7440e1db4cd33/domain/operator/events HTTP/1.1" 200 63
```

On success RDM responds with status 200 and a blob of data confirming the log_id and sequence:
```
> 2021-11-17 13:03:14,430 file_upload DEBUG upload_file:   upload_file returned 200
{
    "log_id": "069ca842-efbb-465c-b0cb-120988ec7acc",
    "seq": 1435662
}
```

#### File System Upload Specific Configuration
The process is the same for the file_system domain except for the various differences in configuration. Take care to correctly specify the RDM type as ```fs::file``` or ```fs::folder``` as appropriate.

```
> 2021-11-17 13:14:52,462 file_upload INFO main:   Running file_upload.py for server=https://us-qa-api.sitelink.topcon.com:443 dc=us site=fdc59e465e0476fbf5bebc1314bf73c23b7ee0c1a095716b57a7440e1db4cd33> 2021-11-17 13:14:52,463 file_upload DEBUG upload_file:   Upload file to https://us-qa-api.sitelink.topcon.com:443/file/v1/sites/fdc59e465e0476fbf5bebc1314bf73c23b7ee0c1a095716b57a7440e1db4cd33/upload
> 2021-11-17 13:14:52,463 file_upload DEBUG upload_file:   File Upload payload: {
    "upload-uuid": "abb390be-4629-4a8e-8fd6-19ddb0afa66d",
    "upload-file-name": "file_to_upload.txt",
    "upload-file-size": 9
}
> 2021-11-17 13:14:52,468 connectionpool DEBUG _new_conn:   Starting new HTTPS connection (1): us-qa-api.sitelink.topcon.com:443
> 2021-11-17 13:14:53,844 connectionpool DEBUG _make_request:   https://us-qa-api.sitelink.topcon.com:443 "POST /file/v1/sites/fdc59e465e0476fbf5bebc1314bf73c23b7ee0c1a095716b57a7440e1db4cd33/upload?upload-uuid=abb390be-4629-4a8e-8fd6-19ddb0afa66d&upload-file-name=file_to_upload.txt&upload-file-size=9 HTTP/1.1" 200 37
> 2021-11-17 13:14:53,845 file_upload DEBUG upload_file:   File RDM payload: {
    "_id": "abb390be-4629-4a8e-8fd6-19ddb0afa66d",
    "name": "file_to_upload.txt",
    "size": 9,
    "_rev": "02766f6e-ac7c-488c-b74d-961030fb767e",
    "_v": 0,
    "_at": 1637118892463,
    "uuid": "abb390be-4629-4a8e-8fd6-19ddb0afa66d",
    "_type": "fs::file"
}
> 2021-11-17 13:14:53,848 file_upload DEBUG upload_file:   Upload RDM to https://us-qa-api.sitelink.topcon.com:443/rdm_log/v1/site/fdc59e465e0476fbf5bebc1314bf73c23b7ee0c1a095716b57a7440e1db4cd33/domain/file_system/events
> 2021-11-17 13:14:54,451 connectionpool DEBUG _make_request:   https://us-qa-api.sitelink.topcon.com:443 "POST /rdm_log/v1/site/fdc59e465e0476fbf5bebc1314bf73c23b7ee0c1a095716b57a7440e1db4cd33/domain/file_system/events HTTP/1.1" 200 63
> 2021-11-17 13:14:54,453 file_upload DEBUG upload_file:   upload_file returned 200
{
    "log_id": "21a31c14-f1d3-498b-b7fa-a1cc35e34e0b",
    "seq": 1435666
}
```
