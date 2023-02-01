# Amazon Pinpoint & GTM integration

### Description
[Amazon Pinpoint](https://aws.amazon.com/pinpoint/) is a multichannel customer engagement platform, which marketers and developers can use to deliver customer communications across six channels, segments, and campaigns at scale. Amazon Pinpoint allows its users to record customer events using its APIs. These events can be used for behavioural analytics or to trigger campaigns and [journeys](https://docs.aws.amazon.com/pinpoint/latest/userguide/journeys.html).

It is very common nowadays for a business to make use of a Tag Manager such as Google Tag Manager (GTM). Tag managers allow you to record customer events from your mobile and web applications and share them with other platforms that require these events  e.g. Google Campaign Manager or Facebook Ads. Before tag managers, if a new platform had to record behavioural events, businesses needed to hardcode its tracking pixel on their web/mobile app whereas now they add this pixel to their tag manager.

Amazon Pinpoint users who want to record customer events from their web or mobile application need to either use [AWS Amplify](https://docs.amplify.aws/lib/analytics/getting-started/q/platform/js/) or call the Amazon Pinpoint API from their backend application. The above might duplicate the recording of these events in case there is already a tag manager e.g. GTM.

### Solution
Currently Amazon Pinpoint doesn't offer a tracking pixel or integration with GTM. The solution in this GitHub repository creates an Amazon API Gateway endpoint that can be used as a [GTM Custom Image Tag](https://support.google.com/tagmanager/answer/6107167?hl=en). GTM Custom Image Tags can append [GTM variables](https://support.google.com/tagmanager/answer/7683362?hl=en) as URL parameters when calling the Custom Image Tag. This way the application sitting in the other side of the API Gateway receives all information required to record a custom event by calling the Amazon Pinpoint API.



### Use-cases

1) Subscription/Contract renewal
2) Appointment reminder

