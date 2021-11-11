# Getting Authorized and Connected to the Sitelink3D v2 API
To use the Sitelink3D v2 API, clients require an Access Token to authorize their requests. On this page, we will walk through the process of obtaining an Access Token to use when making calls to the Sitelink3D v2 API within your application.

Firstly, third-party Sitelink3D v2 Client’s must be registered to use the Sitelink3D v2 API.  Clients are issued a Client Id and Client Secret that are used to authenticate their requests.
To register a new Third-party Client, please submit a request via:

____

To obtain an Access Token, there are two available methods that your application can provide users:

1. For users with a Topcon account; logging in using the Topcon SSO authentication, or
2. For users without a Topcon account; using a Sitelink3D v2 External User Policy Scope ID.

See below for instructions on enabling both methods within your application.

## Method 1. Using Topcon SSO Authentication

> More information about the OAuth Authorization Code flow:
> https://datatracker.ietf.org/doc/html/rfc6749#section-4.1
> We also provide the ability to use PKCE for an extra layer of security:
> https://datatracker.ietf.org/doc/html/rfc7636
> PKCE is required if the client is unable to secure its credentials; desktop and mobile apps. In all other cases PKCE is optional.

### Step 1. Redirect user to authorization page
The user will be required to sign into Topcon SSO and then authorize the connection with the client.

![Sign In](https://github.com/Sitelink3D-v2-Developer/sitelink3dv2-api-documentation/blob/SHTS-3000/setup/images/getting_authorized_sign_in.png "Sign In")

![Request Access](https://github.com/Sitelink3D-v2-Developer/sitelink3dv2-api-documentation/blob/SHTS-3000/setup/images/getting_authorized_request_access.png "Request Access")

#### Request
```https://{host}/authorize?client_id={}&redirect_uri={}&response_type=code&state={}&code_challenge={}&code_challenge_method=s256```

#### Parameters

| Key                          | Value                                                                                                                                       |
|------------------------------|---------------------------------------------------------------------------------------------------------------------------------------------|
| host                         | QA: qa.sitelink.topcon.com Production: sitelink.topcon.com                                                                                  |
| client_id                    | Provided by Topcon when a client is registered                                                                                              |
| redirect_uri                 | Must match the redirect uri provided when the client was registered                                                                         |
| response_type                | "code" is currently the only supported value                                                                                                |
| state                        | Optional. This value will be passed back to the client during the next step                                                                 |
| code_challenge (PKCE)        | Optional. If PKCE is required, a base64 encoded sha256 of a random string                                                                   |
| code_challenge_method (PKCE) | Optional. Indicates the method for validation the challenge, "s256" is expected. "plain" is available if the client can't perform a sha256. |

### Step 2. Handle Redirect
Once the user has authorized the connection they will be redirected to "{redirect_uri}?code={}&state={}"

Upon receiving this redirect the client will make a request to receive the Access Token

#### Request
```Post https://{host}/token?code={}&grant_type=authorization_code&code_verifier={}```

#### Headers
| Key           | Value                                     |
|---------------|-------------------------------------------|
| Authorization | Basic base64(```{client_id}```:```{client_secret}```) |

#### Parameters
| Key           | Value                                                                                                                    |
|---------------|--------------------------------------------------------------------------------------------------------------------------|
| host          | QA: qa.sitelink.topcon.com Production: sitelink.topcon.com                                                               |
| code          | The code that was received in the redirect. Valid for 30 seconds.                                                        |
| grant_type    | authorization_code                                                                                                       |
| code_verifier | Optional: if code_challenge was provided in the previous step, this is the random string used to form the code_challenge |

#### Response

```json
{
    "access_token": "opaque",
    "token_type": "Bearer",
    "expires_in": 86400,
    "refresh_token": "opaque"
}
```

### Step 3. Use Access Token with API requests

Now that you have the Access Token authenticating against the Sitelink3D v2 API is easy - simply add the Access Token to a HTTP header (preferred) or a query parameter.

#### 3.1 HTTP Header Authentication (preferred)
To use the HTTP Header authentication simply add an "Authorization" HTTP header with the value "Bearer {access_token}"

#### 3.2 Query Parameter Authentication
To use query parameter authentication simply add an "access_token" parameter to the request URL

> NOTE: This method should only be used when you are not able to add HTTP headers.

### Step 4. Use Refresh Token to obtain new Access Token
Refresh tokens are valid for 30 days and are single use only.

#### Request
```POST https://{host}/token?refresh_token={}&grant_type=refresh_token```

#### Headers
| Key           | Value                                     |
|---------------|-------------------------------------------|
| Authorization | Basic base64(```{client_id}```:```{client_secret}```) |

#### Parameters
| Key           | Value                                                      |
|---------------|------------------------------------------------------------|
| host          | QA: qa.sitelink.topcon.com Production: sitelink.topcon.com |
| refresh_token | The refresh token from the previous token request          |
| grant_type    | refresh_token                                              |

#### Response
Same as previous token request

## Method 2. Using a Sitelink3D v2 External User Policy Scope ID

### Step 1. Creating an External User

Sign in to the Sitelink3D v2 web portal appropriate for the environment and click Main menu → User Manager

| Environment Name | URL                                          |
|------------------|----------------------------------------------|
| QA               | http://topcon-ace-stg.topconpositioning.com/ |
| Production       | https://sitelink.topcon.com/                 |

![User Manager](https://github.com/Sitelink3D-v2-Developer/sitelink3dv2-api-documentation/blob/SHTS-3000/setup/images/getting_authorized_user_manager.png "User Manager")

Click Manage External Users from the list actions.

![Manage External Users](https://github.com/Sitelink3D-v2-Developer/sitelink3dv2-api-documentation/blob/SHTS-3000/setup/images/getting_authorized_manage_external_users.png "Manage External Users")

Click New External User from the page actions.

![New External User](https://github.com/Sitelink3D-v2-Developer/sitelink3dv2-api-documentation/blob/SHTS-3000/setup/images/getting_authorized_new_external_user.png "New External User")

Enter the new External User details and click Save.

![Save External User](https://github.com/Sitelink3D-v2-Developer/sitelink3dv2-api-documentation/blob/SHTS-3000/setup/images/getting_authorized_save_external_user.png "Save External User")

### Step 2. Assign the External User to Roles

In the User Manager, select the site or the organization that you want grant external access to. Then, select a Role from the Roles list and click Manage Members from the list actions.

![Manage Role Members](https://github.com/Sitelink3D-v2-Developer/sitelink3dv2-api-documentation/blob/SHTS-3000/setup/images/getting_authorized_manage_role_members.png "Manage Role Members")

Finally, select the External User to grant the Role access for the Site and click Save.

![Save Role Members](https://github.com/Sitelink3D-v2-Developer/sitelink3dv2-api-documentation/blob/SHTS-3000/setup/images/getting_authorized_save_role_members.png "Save Role Members")

### Step 3. Copy the External User Policy Scope ID

In the External Users window, select the External User and click View Policy from the list actions.

![View Policy](https://github.com/Sitelink3D-v2-Developer/sitelink3dv2-api-documentation/blob/SHTS-3000/setup/images/getting_authorized_view_policy.png "View Policy")

The External User's Policy Scope ID is displayed in the top left of the window. Click the Clipboard icon to copy it to your clipboard.

![Copy Policy](https://github.com/Sitelink3D-v2-Developer/sitelink3dv2-api-documentation/blob/SHTS-3000/setup/images/getting_authorized_copy_policy.png "Copy Policy")

> NOTE: The External User Policy window also allows you to copy the Resource URL for sites and organizations. The Resource URL contains the site UUID and organization UUID required for creating Sitelink API endpoint URLs.  See the Sitelink API Swagger documentation for more information here: https://api.sitelink.topcon.com/swagger

### Step 4. Exchange the Policy Scope ID for an Access Token
The Sitelink3D v2 API OAuth Token endpoint is used to exchange the Policy Scope ID and some query parameters. It is authenticated with your third-party Client ID and Secret (provided to you as a result of Step 1) for an Access Token.

To do this, send a HTTP POST request to this endpoint

| Environment Name | URL                                               |
|------------------|---------------------------------------------------|
| QA               | https://qa-api.sitelink.topcon.com/oauth/v1/token |
| Production       | https://api.sitelink.topcon.com/oauth/v1/token    |

with the following Query Params:

```grant_type=client_credentials```

```scope=[Paste the Policy Scope ID from Step 4]```

e.g. ```scope=123456789abcdefghi12345678912```

using an "Authorization" HTTP Header with the value of "Basic " + the encoded Base64 string of "[Third-party Client ID]:[Third-party Client Secret]"

e.g. ```Authorization=Basic MTExMTExMTEtMjIyMi0zMzMzLTQ0NDQtNTU1NTU1NTU1NTU1OjExMTExMTExLTIyMjItMzMzMy00NDQ0LTU1NTU1NTU1NTU1NQ==```

The POST result contains the Access Token ```"access_token"``` that can be used to authenticate Sitelink3D v2 API requests.

A python example that performs this exchange is available in our example repository [here](https://github.com/Sitelink3D-v2-Developer/sitelink3dv2-examples/tree/main/components/tokens).

#### Postman REST Client Example
The following example demonstrates the above process in the production environment. Adjust the URL if working with QA.

![Postman Request Token](https://github.com/Sitelink3D-v2-Developer/sitelink3dv2-api-documentation/blob/SHTS-3000/setup/images/getting_authorized_postman_request_token.png "Postman Request Token")

![Postman Response Token](https://github.com/Sitelink3D-v2-Developer/sitelink3dv2-api-documentation/blob/SHTS-3000/setup/images/getting_authorized_postman_response_token.png "Postman Response Token")

#### CURL Example
```
curl --location --request POST 'https://api.sitelink.topcon.com/oauth/v1/token?grant_type=client_credentials&amp;scope=123456789abcdefghi12345678912' --header 'Authorization: Basic MTExMTExMTEtMjIyMi0zMzMzLTQ0NDQtNTU1NTU1NTU1NTU1OjExMTExMTExLTIyMjItMzMzMy00NDQ0LTU1NTU1NTU1NTU1NQ=='
```

### Step 5. Use Access Token with API requests
Now that you have the Access Token, authenticating against the Sitelink3D v2 API is easy - simply add the Access Token to a HTTP Header (preferred) or a Query Parameter.

#### 5.1. HTTP Header Authentication (Preferred)
To use HTTP Header authentication, simply add an “Authorization” HTTP Header with value “Bearer {access_token}".

Postman REST Client example:
![Postman Request Header](https://github.com/Sitelink3D-v2-Developer/sitelink3dv2-api-documentation/blob/SHTS-3000/setup/images/getting_authorized_postman_request_header.png "Postman Request Header")

#### CURL Example
```
curl --location --request GET "https://api.sitelink.topcon.com/siteowner/v1/sites/8ff109c4-d56c-49d0-97e6-0057d4e9f366/site_config" --header "Authorization: Bearer iIbU_tiTy2DaO9U6ZZxYbjXIRVO2.u5eOAQqt2qnJKZO7fIYzuW4jTmOTECN--Nv5tEJjQdy0-WI2GaEIM5MOgxwMDnogMyt4LQUNTsiIhQ1_DRsIyi4p3lMRtyWHIlDp0J8BghWN2YcMLeiiv75KfALyC.xRcNEyfliCAQhQETeqyO6gkicbjldjrGpKdJVoxF9hMvwsNMJjG75"
```

#### 5.2. Query Parameter Authentication
To use query parameter authentication, simply add an “access_token” parameter to the request URL as per the below examples. This method should only be used when you are not able to add HTTP Headers.

#### Postman REST Client Example
![Postman Request Params](https://github.com/Sitelink3D-v2-Developer/sitelink3dv2-api-documentation/blob/SHTS-3000/setup/images/getting_authorized_postman_request_params.png "Postman Request Params")

#### CURL Example
```
curl --location --request GET "https://api.sitelink.topcon.com/siteowner/v1/sites/8ff109c4-d56c-49d0-97e6-0057d4e9f366/site_config?access_token=iIbU_tiTy2DaO9U6ZZxYbjXIRVO2.u5eOAQqt2qnJKZO7fIYzuW4jTmOTECN--Nv5tEJjQdy0-WI2GaEIM5MOgxwMDnogMyt4LQUNTsiIhQ1_DRsIyi4p3lMRtyWHIlDp0J8BghWN2YcMLeiiv75KfALyC.xRcNEyfliCAQhQETeqyO6gkicbjldjrGpKdJVoxF9hMvwsNMJjG75"
```



