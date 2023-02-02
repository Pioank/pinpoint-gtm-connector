# Amazon Pinpoint & GTM integration

### Description
[Amazon Pinpoint](https://aws.amazon.com/pinpoint/) is a multichannel customer engagement platform, which marketers and developers can use to deliver customer communications across six channels, segments, and campaigns at scale. Amazon Pinpoint allows its users to record customer events using its APIs. These events can be used for behavioural analytics or to trigger campaigns and [journeys](https://docs.aws.amazon.com/pinpoint/latest/userguide/journeys.html).

It is very common nowadays for a business to make use of a Tag Manager such as Google Tag Manager (GTM). Tag managers allow you to record customer events from your mobile and web applications and share them with other platforms that require these events  e.g. Google Campaign Manager or Facebook Ads. Before tag managers, if a new platform had to record behavioural events, businesses needed to hardcode its tracking pixel on their web/mobile app whereas now they add this pixel to their tag manager.

Amazon Pinpoint users who want to record customer events from their web or mobile application need to either use [AWS Amplify](https://docs.amplify.aws/lib/analytics/getting-started/q/platform/js/) or call the Amazon Pinpoint API from their backend application. The above might duplicate the recording of these events in case there is already a tag manager e.g. GTM.

### Use-cases
1) Trigger campaign/journeys based on behavioural events that you are already recording using GTM
2) Update customer profiles based on the latest events tracked from GTM

### Solution
Currently Amazon Pinpoint doesn't offer a tracking pixel or integration with GTM. The solution in this GitHub repository creates an Amazon API Gateway endpoint that can be used as a [GTM Custom Image Tag](https://support.google.com/tagmanager/answer/6107167?hl=en). GTM Custom Image Tags can append [GTM variables](https://support.google.com/tagmanager/answer/7683362?hl=en) as URL parameters when calling the Custom Image Tag. This way the application sitting in the other side of the API Gateway receives all information required to record a custom event by calling the Amazon Pinpoint API.

![architecture_diagram](https://github.com/Pioank/pinpoint-gtm-connector/blob/main/Assets/ArchitectureDiagram-Pinpoint-GTM.PNG)

The above architecture uses Amazon API Gateway to create an endpoint, Amazon SQS to batch & queue the click-stream events and AWS Lambda to process them and record the respective custom events to Amazon Pinpoint. Note that GTM is performing a GET request to the Amazon API Gateway endpoint, thus all data are passed as URL parameters and not in the headers or request body.

### URL parameters structure
This solution utilizes URL parameters to retrieve the required event data from GTM to Pinpoint. The required URL parameters from Amazon Pinpoint [put_events API operation](https://docs.aws.amazon.com/pinpoint/latest/apireference/apps-application-id-events.html) are **event** and **endpoint_id**. Along with the above URL parameters, put_events also requires a **Timestamp** that is generated in the AWS Lambda function but you can configure the solution to retrieve it as a URL parameter for higher accuracy.

Amazon Pinpoint's put_events API operation can also include event attributes e.g. Value, ProductType as well as update the endpoint/user attributes e.g. FirstName or PurchaseDate. To add event attributes and user attributes, the GTM custom image tag URL parameters should start with **evat_{event-attribute}**  for event attributes and **usat_{user-attribute}** for user attributes. The application scans all URL parameters and if the first four characters match with **evat** or **usat**, it populates the API request body accordingly.

**Example of Image URL in GTM custom image tag:**

https://xxxxx.execute-api.us-east-1.amazonaws.com/Api-PinpointEvents/events?event={{Event}}&endpoint_id={{endpoint_id}}&evat_value={{value_dollars}}&usat_purchase_date={{purchase_date}}

### Authentication
As seen above, Amazon Pinpoint's put_events API operation can also update the endpoint/user attributes. Updating customer data without any authentication can have catastrophic results as this data is used for customer segmentation and message personalization purposes. At the same time there might be a requirement to record behavioural events from unauthenticated users (not logged-in).

**Scenario 1 - Authenticated users:** Customers will need to authenticate to update customer data in Amazon Pinpoint. Amazon API Gateway offers a variety of ways to authorize your requests including Amazon Cognito user pools, Lambda authorizers etc. - see full list [here](https://docs.aws.amazon.com/apigateway/latest/developerguide/apigateway-control-access-to-api.html). Note unauthenticated customers can still have their events recorded but they won't be able to update their customer data in Amazon Pinpoint.

**Scenario 2 - Unauthenticated users:** Customers' events will be recorded without the need to authenticate. Updating customer data can be enabled or restricted to specific attributes.

In both scenarios, it is recommended to make use of [cross-origin resource sharing](https://en.wikipedia.org/wiki/Cross-origin_resource_sharing)(CORS) to restrict resources on a web page to be requested from another domain outside the domain which the first resource was served, see how to enable CORS for an Amazon API Gateway REST API resource [here](https://docs.aws.amazon.com/apigateway/latest/developerguide/how-to-cors.html)

### Considerations
The Amazon API Gateway authentication is not offered as part of this solution

### Prerequisites
1) AWS account
2) Amazon Pinpoint project
3) At least one Amazon Pinpoint channel enabled and one endpoint under that channel
4) A Google Tag Manager account

### Implementation
1. Create or use an existing Amazon S3 bucket, which will be used to host the AWS Lambda code
2. Upload the [AWS Lambda code zipped file](https://github.com/Pioank/pinpoint-gtm-connector/blob/main/PinpointPixelTracking.zip) to the S3 bucket created or selected on the step above
3. Deploy the [AWS CloudFormation template](https://github.com/Pioank/pinpoint-gtm-connector/blob/main/PinpointGTM-CFtemplate.yaml) 
4. Create a [Google Tag Manager Space](https://support.google.com/tagmanager/answer/7059647?hl=en) or access an existing one.
5. From the **Built-In Variables** select **Event** and for the **User-Defined Variables** create one with the name **endpoint_id** and **Type** - **Data Layer Variable**
![GTM_BuiltInVariables](https://github.com/Pioank/pinpoint-gtm-connector/blob/main/Assets/GTM_BuiltInVariables.png)
6. Create a new **Trigger** with **Event name** - **purchase_success**. Select **All Custom Events** as **This trigger fires on**
![GTM_Triggers](https://github.com/Pioank/pinpoint-gtm-connector/blob/main/Assets/GTM_Triggers.png)
7. Create a new **Tag** with **Tag Type = Custom Image** and for **Image URL** paste the API Gateway's endpoint with the required URL parameters - see  example:https://xxxxxx.execute-api.us-east-1.amazonaws.com/Api-PinpointEvents/events?**event={{Event}}&endpoint_id={{endpoint_id}}**. Under **Firing Triggers** select **purchase_success** custom event
![GTM_PinpointTag](https://github.com/Pioank/pinpoint-gtm-connector/blob/main/Assets/GTM_PinpointTag.png)

### Testing
1. Download the [demo HTML page](https://github.com/Pioank/pinpoint-gtm-connector/blob/main/Assets/Pinpoint_GTM-HTML-Template.html) and replace LINE 12 & 18 with the code provided by GTM
![GTM_Tags](https://github.com/Pioank/pinpoint-gtm-connector/blob/main/Assets/GTM_Tags.png)
2. Create an Amazon Pinpoint jounrey and make sure that the **Event** name under the **Entry** activity is written the same as in GTM and when loading the demo HTML page.
![Pinpoint-journey](https://github.com/Pioank/pinpoint-gtm-connector/blob/main/Assets/Pinpoint_Journey.png)
3. Once the Amazon Pinpoint journey is published, open the demo HTML page on your local machine using a browser of your preference
4. Type the endpoint id and event name and select **Submit**. This will record an event on GTM, which will then be forwarded to the GTM Custom Image tag => API Gateway endpoint => AWS Lambda => Amazon Pinpoint
![HTML-site](https://github.com/Pioank/pinpoint-gtm-connector/blob/main/Assets/HTML-site.PNG)


