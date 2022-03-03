# Creating a Topcon Account and Organization
This page introduces the Sitelink3D v2 environment to new users. The process is comprised of creating an account and subsequently an organization. The purpose of this process is primarily to facilitate the development of applications using the Sitelink3D v2 API. This process does not currently require payment.

> **_NOTE:_**  Two distinct Siteilnk3D v2 environments exist. Topcon recommends creating an account and one or more organizations on both environments to ensure adequate isolation of development and production activity.

## Sign Up
1. In a browser window, go to the URL appropriate for the environment and click “Sign Up” in the top right corner.

| Environment Name | URL                                          |
|------------------|----------------------------------------------|
| QA               | http://topcon-ace-stg.topconpositioning.com/ |
| Production       | https://sitelink.topcon.com/                 |

![Sign Up](https://github.com/Sitelink3D-v2-Developer/sitelink3dv2-api-documentation/blob/master/setup/images/creating_account_sign_up.png "Sign Up")

2. Fill out the form and click “Create Account”.

![Create Account](https://github.com/Sitelink3D-v2-Developer/sitelink3dv2-api-documentation/blob/master/setup/images/creating_account_create_account.png "Create Account")

3. Check the email address you provided and click on the confirmation link to activate your account.

![Create Account Success](https://github.com/Sitelink3D-v2-Developer/sitelink3dv2-api-documentation/blob/master/setup/images/creating_account_create_account_success.png "Create Account Success")

![Create Account Email](https://github.com/Sitelink3D-v2-Developer/sitelink3dv2-api-documentation/blob/master/setup/images/creating_account_create_account_email.png "Create Account Email")

## Sign In
1. Click the “Sign In” button from the activation page or click “Log In” at the top right. Then, enter the email address and password created above and click "Sign In".

![Sign In](https://github.com/Sitelink3D-v2-Developer/sitelink3dv2-api-documentation/blob/master/setup/images/creating_account_create_account_sign_in.png "Sign In")

## Create an Organization
1. Once signed in, click on your user name at the top right and select “Create Organization”.

![Create Organization](https://github.com/Sitelink3D-v2-Developer/sitelink3dv2-api-documentation/blob/master/setup/images/creating_account_create_organization.png "Create Organization")

2. Populate the required fields.

![Create Organization Form](https://github.com/Sitelink3D-v2-Developer/sitelink3dv2-api-documentation/blob/master/setup/images/creating_account_create_organization_form.png "Create Organization Form")

3. Submit the form to complete the creation of your organization.

![Create Organization Success](https://github.com/Sitelink3D-v2-Developer/sitelink3dv2-api-documentation/blob/master/setup/images/creating_account_create_organization_success.png "Create Organization Success")

## Access Sitelink3D v2 Web Portal
Open the Sitelink3D v2 web portal appropriate for the environment. You are now logged in. Your organization is visible in the menu under the user name in the top right. Note that the Sitelink3D v2 link under the My Apps menu at the top left of the Topcon Blue Bar will not yet be visible as Service Points have yet to be added to the organization.

| Environment Name | URL                             |
|------------------|---------------------------------|
| QA               | https://qa.sitelink.topcon.com/ |
| Production       | https://sitelink.topcon.com/    |

![View Organization](https://github.com/Sitelink3D-v2-Developer/sitelink3dv2-api-documentation/blob/master/setup/images/creating_account_view_organization.png "View Organization")

## Obtain Service points
Many Sitelink3D v2 functions, including the creation and localization of sites, will be openly available without the need for Service Points on both environments. To start connecting devices and pushing data however, Service Points will be required. There are obtained as follows.

| Environment Name | Procedure                                                                                                           |
|------------------|---------------------------------------------------------------------------------------------------------------------|
| QA               | Contact sitelink3d-api-support@topcon.com with your QA Organization name to request Service Points for development. |
| Production       | Contact you Topcon Dealer to purchase Service Pointes in blocks of 500.                                             |
