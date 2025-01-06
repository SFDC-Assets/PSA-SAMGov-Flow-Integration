![Public Sector Accelerators logo](/docs/Logo_GPSAccelerators_v01.png)

# SAM.gov Integration with Flow and External Service
**Provides a no-code integration with SAM.gov for querying and retrieving verified entity information.**

View: [Accelerator Site Listing](https://pubsec-accelerators.my.site.com/accelerators/accelerator/a0wDo000002494qIAA/samgov-integration-with-flow-and-external-service)


## Description

This Accelerator allows you to integrate Salesforce with SAM.gov using native, no-code capabilities.  With it, you can call the [System for Award Management (SAM.gov) Entity Management API](https://open.gsa.gov/api/entity-api/).  The Accelerator includes an example using Salesforce Flow that searches for an entity using a CAGE Code and returns commonly-used data elements.

This integration uses the Salesforce Platform's native capabilities for calling an external data source (SAM.gov) using an [External Service](https://help.salesforce.com/s/articleView?id=sf.external_services.htm&type=5). This service can be used by Salesforce in no/low-code (e.g. Flow) and pro-code (e.g. Apex) ways.

> [!CAUTION] 
> - Use of the SAM.gov API requires that you register with the [SAM.gov website](https://sam.gov/content/home) through your organization, request the appropriate role(s), and request an API key. *This package does not include an API key; you must provide your own via the post-installation configuration in order to use this capability.*
> - There are [API key rate limits](https://open.gsa.gov/api/entity-api/#api-key-rate-limits) for the Entity Management endpoint which are dependent upon your registered account. Additionally, Salesforce organizations have their own [API Request Limits and Allocations](https://developer.salesforce.com/docs/atlas.en-us.salesforce_app_limits_cheatsheet.meta/salesforce_app_limits_cheatsheet/salesforce_app_limits_platform_api.htm). This project **does not** provide a mechanism for tracking your usage against these limits. *By using this package, you assume all responsibility for tracking your API callouts and staying within the limits of both SAM.gov and Salesforce.*

## Included Assets

This Accelerator includes the following assets:
1. An **unmanaged package** (link below; metadata is also found in the /force-app/main/default/ folder) that includes:
    - Custom Metadata Type definition (x1)
    - Custom Metadata Type record (x1)
    - External Credential (x1)
    - Named Credential (x1)
    - External Service (x1)
    - Flow (x1)
    - Permission Set (x1)
2. **Documentation**, including:
    - This readme file

## Before You Install

### License Requirements

- Salesforce Platform (or higher)

### General Assumptions

While this package is designed to be largely plug-and-play and details have been added to expand on why certain design patterns were used, there are a few assumptions to call out:
- You are using this Accelerator in a sandbox or dev environment for development and testing prior to promoting to production. It is recommended that you not install any Accelerator directly into production environments.
- You are familiar with Salesforce Administration, especially with regard to permission sets and profile management.
- You are familiar with Salesforce Flow and best practices to extend and reuse the included example flow.
- You are familiar with basic REST API concepts such as authentication mechanisms and GET methods.

## Installation

There are no dependencies for installing this package. Use [this package link](https://login.salesforce.com/packaging/installPackage.apexp?p0=04tWs000000GD9N) to access the installer and complete the authentication process for your destination org.
1. Choose install for Admins only
2. Click the Install button to complete the installation process

> [!NOTE]
> You may see a message "This app is taking a long time to install." Click Done and wait for an email letting you know the package has been installed successfully. Wait until receiving this email before continuing with the post-install configuration.

## Post-Install Setup & Configuration

### Base Configuration

In order to use the External Service:
1. Assign the SAM.gov API Access permission set to users or Permission Set Groups
2. Update the SAM.gov Configuration Custom Metadata Type record with your own API key
    1. Go to Setup, Custom Metadata Types
    2. Click Manage Records next to SAM.gov Configuration
    3. Click the *Default* record label to access the detail page
    4. Click Edit and update your *api_key* value
    5. Click Save to confirm your changes

### Create a Parent Flow

This Accelerator has included an example autolaunched flow (SAM.gov Entity Search - CAGE Code) using the External Service to search for an entity based on the organization's CAGE Code, and returns certain information from the API response about the entity back to the parent flow. You can test this flow after completing the above base configuration steps by opening the latest version in the Flow Builder and using the Debug option to enter a cageCode value. 
1. From the Automation app or the Setup menu, navigate to the SAM.gov Entity Search - CAGE Code flow and open the latest version
2. Click the Debug button in the top-right
3. Enter a value in the cageCode field under Input Variables and click Run
4. A successful callout should follow the entire "Query Success" path and have data in the `samGovEntityRegistrationValues` collection (the flow's output or return variable) which can be seen by clicking the accordion next to the "Assigment: Set Transform Collection for Output" step of the Debug Details pane on the right

> [!IMPORTANT]
> Do note that even in the instances where you expect only one response from the search (i.e. searching by cage or UEI number) the API always returns a collection.

To properly use this autolaunched flow, you'll want to create a parent flow to call it and handle the resulting information. You can do this by creating a simple screen flow to get an understanding of how the data flows into and back from the subflow.
1. Create a new flow from the Automation app (or from the Flow menu in Setup)
2. Choose Start From Scratch when given the choice in Flow Builder and click Next
3. Select Screen Flow and click Create
4. Click the + under the start element and then click Screen to add a new screen element to the canvas and give the screen a label and API name
5. Drag and drop a Text component onto the screen which will hold our cage code and give it a label and API name and then click Done
6. Click the + under the screen element you just added and choose Subflow to add an autolaunched flow action to the canvas
7. Search for "SAM.gov" to find the SAM.gov Entity Search - CAGE Code flow included in the package, click it, and give it a label and API name. Click the cageCode toggle to include our input value and, in the lookup box, choose the screen element from before and then the text component for our cage code
8. Click the + under the subflow element and then click Decision to add conditional logic to your flow and give it a label and API name
    1. Add a label and API name for a 'successful' outcome. The example flow returns not only the results from the API callout, but a boolean value of true if it receives a [200 response code](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/200)
    2. Leave the condition requirements as All conditions are met and, in the Resource lookup box, choose the Outputs from the subflow, then the samGovCalloutSuccess value
    3. Leave the operator as Equals and choose True from the Value lookup box under global constants
    4. Click on the Default Outcome and give it an appropriate label to handle a failed callout (e.g. "Failure")
9. Click the + under the successful outcome branch and click Loop to add a loop element to the canvas and give it a label and API name
    1. Under Select Collection Variable, choose the Outputs from the subflow, then the samGovEntityRegistrationValues variable
10. Click the + under the For Each branch of the loop and click Screen to add another screen element to the canvas and give it a label and API name
11. Drag and drop a Text component onto the screen which will hold a response value from the callout and give it an appropriate label and API name
    1. For the Default Value of the text component, choose Current Item From Loop under the Apex-defined Variables, then choose the legalBusinessName option
    2. Optionally set the Disabled option to True under global constants to ensure the value from the callout cannot be edited
12. Repeat the previous step (11) as desired to include values for any of the other available data points; Click Done once you have added the values you want to display.
    * Optionally, instead of using Text components, you can also display the data in a Display Text component with rich text formatting
    * In a real-world scenario, you would want to build your flow and user interfaces with an understanding of what the expected query results will be, and how the information would need to be used throughout the rest of the flow. For instance, if the resulting information is needed to create or update records, you would need to use the Assignment element to move to values from the Current Item From Loop variables to something more permanent
13. Click the + under the failure outcome branch of your decision element created in step (8) and then click Screen to add a new screen element to the canvas and give it a label and API name
14. Drag and drop a Display Text component onto the screen which will hold the failure response and give it an API name
15. Choose the Outputs from the subflow, then the failureResponse value in the Insert a resource lookup box. If something should go wrong with the subflow, this will allow you to gracefully handle the outcome
16. Click Save to save your flow, give it a label and API name, and then click Debug to test your flow

The above is merely an example to display information on a screen given a user's input. There are additional steps needed to take the information returned and use it to create a new record or update an existing record, and to utilize data from queries returning multiple results (such as searching by the legal name). You could also use this design pattern of calling a subflow from a record-triggered flow to automatically run a validation against the entity management API when a record is created or updated, although there are other considerations which require thoughtful review. Impacts to performance, the use of asynchronous processing, an understanding of batch create or update sizes, and overall API consumption on both the SAM.gov and Salesforce sides are some of the things to consider when architecting your solutions to meet business requirements.

### Using This Callout With Experience Cloud Guest Users

Experience Cloud Guest User records cannot be granted access via a permission set to the necessary external credential. In order to allow access, you must go to the profile for the site's Guest User and edit the appropriate permissions:
1. Grant Read access to the User External Credentials object in the Object Settings of the profile
2. Grant access to *samGovApiKey - All Users* in the External Credential Principal Access options
3. Grant access to any parent Flows you create with calls to the included example SAM.gov Entity Search - CAGE Code subflow or those you create yourself. Action must be taken to edit the access options before granting access through profiles:
    1. From the Flow menu, click the action menu for the parent flow you created and choose *Edit Access*
    2. Check the box next to "Override default behavior and restrict access to enabled profiles or permission sets."
    3. Select and add the correct profile(s) for the Guest Users and sites who need access

> [!IMPORTANT]
> Be sure to follow best practices for Flow security. See the [Additional Resources](#additional-resources) section for important insights.
  
## Additional Details
  
### Custom Metadata Type - Explained

This project uses a Custom Metadata Type to store the API key for the SAM.gov service. This is to avoid hard-coding the API key value into any subflows you wish to create and run. Additionally, the SAM.gov API key requires regular refreshes. Using a Custom Metadata Type also enables easier maintenance over time as no changes are required to the flow(s) which utilize the External Service. As another benefit, this design pattern of using a Custom Metadata Type record allows you to easily deploy the same key value between different Salesforce environments without having to run a post-deployment script to write the key to a custom setting or to a custom field on an object.

> [!WARNING]
> If you are using a source-driven development approach, be sure to remove the value of the key in the metadata type record (`customMetadata` > `SAM_gov_Configuration.Default.md-meta.xml`) before posting to any *public* repositories to protect the secrecy of your key.

### External Credential - Explained

Salesforce enables the use of Named Credentials and External Credentials for authentication to simplify and secure callouts to external systems. Permission sets or profiles can be used to grant only specific users access to such endpoints. Although SAM.gov's Entity Management is not an authenticated endpoint (the API key is provided as a URL parameter in the GET method callout), a No Authentication external credential within Salesforce is still used such that access can be granted to the proper user base.

### Permission Set Assignment - Explained

Use the included permission set to provide internal users with access to the endpoint. This grants access to read the External Credentials object and expressed permissions to the SAM.gov external credential.

### External Service - Explained

External Services are a mechanism for configuring, making, and consuming remote calls to external APIs. Some APIs, like the SAM.gov Entity Management endpoint, provide a number of optional parameters which you can use to query for information. Only some of the more common search criteria have been included as part of the configuration for the External Service in this Accelerator:
- legalBusinessName
- cageCode
- dbaName
- dodaac
- ueiSAM

> [!NOTE]
> See the [SAM.gov Entity Management API documentation](https://open.gsa.gov/api/entity-api/) for the full list of optional query parameters.

If your organization needs additional query parameters added to the configuration, edit the External Service to include those options:
1. Go to External Services in the Setup menu
2. Click on the samGovEntitySearch External Service Name
3. Click the action menu for the Search Entity operation and choose Edit HTTP Callout Action
4. Click the + Add Key button under the Set Query Parameter Keys accordion to add a new query parameter. Repeat this step as necessary to add additional parameters.
5. Click Next and follow the on-screen prompts to complete the setup.
    - If you choose Connect for Schema, you will need your API key for this and an example data point.
    - You can also run a query using curl in a command line interface or a software tool like Postman and copy the result if you choose Use Example Response.

## Potential Use Cases
There are a number of different scenarios for which you can use this integration with SAM.gov. These are just a few examples of various levels of complexity, but you can use similar patterns, and extend the External Service using the instructions above, to bring your own ideas to life.
- Forms entry: Use the External Service to simplify data entry and ensure required data is included and correct
    - E.g. Take a parameter like a CAGE Code and automatically fill in the required data fields based on the response to simplify an application process in a screen flow
- Automated validation: Use the External Service to verify information against SAM.gov and update related fields
    - E.g. Run a record-triggered flow when a new Account is created to automatically check and update a custom SAM Registered status field, congressional district, address, or other data on the record
- Entity search against requirements: Use the External Service to query against procurement requirements tracked in Salesforce to find potential partners
    - E.g. Invoke a callout from a bespoke Lightning Web Component to query one or more keys at a time to build a list of potential vendors, such as for Woman Owned Small Business for specific NAICS codes

## Additional Resources

- [Building Secure Screen Flows For External User Access](https://medium.com/salesforce-architects/building-secure-screen-flows-for-external-user-access-0d925cecf564)
- [Building Flows for the Guest User Profile](https://medium.com/salesforce-architects/building-flows-for-the-guest-user-profile-19729b39208a)
- [Record-Triggered Automation | Salesforce Architects](https://architect.salesforce.com/decision-guides/trigger-automation#Asynchronous_Processing)
- [Access Business Processes with External Services | Salesforce Trailhead](https://trailhead.salesforce.com/content/learn/trails/access-business-processes-with-external-services)
- [External Services Superbadge Unit | Salesforce Trailhead](https://trailhead.salesforce.com/content/learn/superbadges/superbadge-external-services-sbu)
- [Distribute and Implement Flows | Salesforce Trailhead](https://trailhead.salesforce.com/content/learn/trails/distribute-and-implement-flows)

## Revision History

**1.1 Winter 2024 release (12 Nov 2024)** - Includes the External Service and related configurations with an example subflow for searching entities by CAGE Code.

## Terms of Use

Thank you for using Global Public Sector (GPS) Accelerators.  Accelerators are provided by Salesforce.com, Inc., located at 1 Market Street, San Francisco, CA 94105, United States.

By using this site and these accelerators, you are agreeing to these terms. Please read them carefully.

Accelerators are not supported by Salesforce, they are supplied as-is, and are meant to be a starting point for your organization. Salesforce is not liable for the use of accelerators.

For more about the Accelerator program, visit: [https://gpsaccelerators.developer.salesforce.com/](https://gpsaccelerators.developer.salesforce.com/)
