# Amazon Pinpoint ISV Partner Guide

This repository serves as a guide for how Independent Software Vendors (ISVs) and other integration partners can build integrations between their product/service and Amazon Pinpoint.  It showcases common integration patterns that Amazon Pinpoint can be integrated with sample use-cases and examples.

The below guide provides examples and patterns that an ISV could use to integrate their application with Amazon Pinpoint.  Use the links below to find pages specific to the ISVs listed:
* [Customer Data Platform (CDP) ISVs](cdp/)
* [Application Development Framework ISVs](app/)
* [Custom Channel ISVs](channel/)
* [Identity Management ISVs](identity/)

## What is Amazon Pinpoint
Amazon Pinpoint is a multi-channel digital engagement service. It is a part of the Customer Engagement suite of services, enabling customers to send both promotional and transactional messages across email, SMS, push notification, voice, and custom channels.  For more details, and a quick guide to Pinpoint terms, see [Amazon Pinpoint Key Concepts](pinpoint_detail/README.md).

See the [Other Amazon Pinpoint Resources](#Other-Amazon-Pinpoint-Resources) below for official documentation, reference architectures with full CloudFormation source code, fully-vetted AWS Solutions, and recorded webinars and other videos.

## Note on Below Integration Patterns

Below is a set of common integration patterns for Amazon Pinpoint.  This is not an exhaustive list of integration patterns.  Customers and partners [can use the APIs](https://docs.aws.amazon.com/pinpoint/latest/apireference/welcome.html) to build a variety of other integrations. This repo's example source code is written in Python and shown using the [AWS Python Boto3 SDK](https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/pinpoint.html).  Integrations can choose to use one of the [AWS SDKs](https://aws.amazon.com/tools/) or call the [REST APIs](https://docs.aws.amazon.com/pinpoint/latest/apireference/welcome.html) directly.  No SDK is required when integrating with Amazon Pinpoint.

## Pattern: Send a message using Amazon Pinpoint
At its core, Amazon Pinpoint is a message delivery service.  ISVs can use Amazon Pinpoint to deliver messages to end-users across email, SMS, push, and voice channels. Amazon Pinpoint uses Amazon Simple Email Service to deliver email messages.  Amazon Pinpoint can send SMS messages in [over 200 countries and regions](https://docs.aws.amazon.com/pinpoint/latest/userguide/channels-sms-countries.html). Amazon Pinpoint supports push messages using Firebase Cloud Messaging, Apple Push Notification service, Baidu Cloud Push, and Amazon Device Message.

The simplest way an ISV can integrate with Amazon Pinpoint to send a message is via the [Send Messages API](https://docs.aws.amazon.com/goto/WebAPI/pinpoint-2016-12-01/SendMessages).  The API supports messages being sent via Email, SMS, Push, and Voice.  Messages can be specified in the API call or abstracted by using re-usable [Amazon Pinpoint Templates](https://docs.aws.amazon.com/pinpoint/latest/userguide/messages-templates.html).  ISVs can specify the address or have Amazon Pinpoint lookup the address by specifying an Endpoint ID.  Further, messages sent via the Send Messages API can also utilize token replacement for [message personalization](https://docs.aws.amazon.com/pinpoint/latest/userguide/message-templates-personalizing.html).

##### Example:  Using the Send Messages API to send an email to 3 recipients using a Template
```python
import boto3
client = boto3.client('pinpoint')

response = client.send_messages(
  ApplicationId='[PinpointProjectId]',
  MessageRequest={
    'Addresses': {
      'success+1@simulator.amazonses.com': {
        'ChannelType': 'EMAIL',
        'Substitutions': {
          'FirstName': ['John']
        }
      },
      'success+2@simulator.amazonses.com': {
        'ChannelType': 'EMAIL',
        'Substitutions': {
          'FirstName': ['Ryan']
        }
      },
      'success+3@simulator.amazonses.com': {
        'ChannelType': 'EMAIL',
        'Substitutions': {
          'FirstName': ['Dave']
        }
      }
    },
    'MessageConfiguration': {
      'EmailMessage': {
        'FromAddress': 'no-reply@example.com'
      }
    },
    'TemplateConfiguration': {
      'EmailTemplate': {
        'Name': 'MyEmailTemplate'
      }
    }
  }
)
```

##### Example:  Using the Send Messages API to send an SMS message with an inline message to a stored endpoint
```python
import boto3
client = boto3.client('pinpoint')

response = client.send_messages(
  ApplicationId='[PinpointProjectId]',
  MessageRequest={
    'Endpoints': {
      '[UniqueEndpointID1ForUser]': {}
    },
    'MessageConfiguration': {
      'SMSMessage': {
        'Body': '{{User.UserAttributes.FirstName}} your one time password is: {{OTP}}',
        'MessageType': 'TRANSACTIONAL',
        'OriginationNumber': '[ORIGINATION_NUMBER]',
        'Substitutions': {
            'OTP': ['123456']
        }
      }
    }
  }
)
```

## Pattern: Send Users and Endpoints to Amazon Pinpoint
ISVs that manage users and user addresses can sync this data to Amazon Pinpoint. This can be used to help build marketing campaign audiences, by providing new users and addresses as they are created, or by augmenting the current set of users and addresses in Amazon Pinpoint by enriching with new data attributes.  These attributes can then be used for message personalization, such as `First Name` or `Item Purchased`, or for filtering when creating dynamic segments to create specific audiences, such as `High Value Customer`, `Users Nearby` or `Newsletter Subscription Status`.

There are multiple different ways that both Endpoints and Users can be sent to Amazon Pinpoint. There is an API to [Update Endpoints](https://docs.aws.amazon.com/pinpoint/latest/developerguide/audience-define-endpoints.html) either [one by one](https://docs.aws.amazon.com/pinpoint/latest/apireference/apps-application-id-endpoints-endpoint-id.html#apps-application-id-endpoints-endpoint-idput) or in [Batch](https://docs.aws.amazon.com/pinpoint/latest/apireference/apps-application-id-endpoints.html#apps-application-id-endpointsput).  Endpoints and users can also be imported in bulk by first [uploading a CSV or JSON file](https://docs.aws.amazon.com/pinpoint/latest/developerguide/audience-define-import.html) to Amazon S3 and [starting an import job](https://docs.aws.amazon.com/pinpoint/latest/apireference/apps-application-id-jobs-import.html#apps-application-id-jobs-importpost) via the APIs.  Lastly, endpoints can also be added and updated when calling the [Events API](https://docs.aws.amazon.com/pinpoint/latest/apireference/apps-application-id-events.html#apps-application-id-eventspost) to submit new user events.

##### Example: Using the Update Endpoint API to update a custom attribute on an endpoint
```python
import boto3
client = boto3.client('pinpoint')

response = client.update_endpoint(
  ApplicationId='[PinpointProjectId]',
  EndpointId='[UniqueEndpointID]',
  EndpointRequest= {
    'Attributes': {
      'MyCustomAttribute': ['Custom Attribute Value 1', 'Custom Attribute Value 2']
    }
  }
)
```

##### Example: Using the Create Import Job API to import a CSV file into Amazon Pinpoint in bulk from Amazon S3
```python
import boto3
client = boto3.client('pinpoint')

response = client.create_import_job(
    ApplicationId='[PinpointProjectId]',
    ImportJobRequest={
        'DefineSegment': False,
        'Format': 'CSV',
        'RoleArn': 'arn:aws:iam::account-id:role/role-name-with-path',
        'S3Url': 's3://bucket-name/folder-name/file-name.csv'
    }
)
```
A full deployable reference architecture that includes Amazon S3 triggers to automatically initiate an Amazon Pinpoint import up on new file dropped in S3 can be found on the [Digital User Engagement Reference Architectures GitHub Repo](https://github.com/aws-samples/digital-user-engagement-reference-architectures#Amazon-S3-Triggered-Endpoint-Imports).


## Pattern: Send User Events to Amazon Pinpoint

ISVs that that track users and their behaviors can send user event data to Amazon Pinpoint.  This can be used to inform Amazon Pinpoint about engagement events that occur from other systems for engagement reporting.  These events can also be used to trigger Amazon Pinpoint Campaigns and Journeys in real-time.  Forgotten password events could be sent from a website login that trigger a campaign to send out a one-time password email.  Item added to cart events could be used to trigger the start of an abandon cart journey.

The [PutEvents API](https://docs.aws.amazon.com/pinpoint/latest/apireference/apps-application-id-events.html#apps-application-id-eventspost) can be used to submit events one at a time or in batches.  Events are tied to endpoints, so a single event that affects 100 endpoints would be submitted as a batch of 100 endpoint events.  Event attributes can be filtered when creating Campaigns or Journeys allowing marketers to select very specific event criteria.

The same API call can be used to update Endpoints at the same time.  This allows for updating endpoint or user attributes at the same time an event is being reported.

##### Example: Calling the PutEvents API with a communication preference update event that also updates the endpoint attributes
```python
import boto3
import time
import datetime
client = boto3.client('pinpoint')

response = client.put_events(
  ApplicationId='[PinpointProjectId]',
  EventsRequest={
    'BatchItem': {
      '[UniqueEndpointID1ForUser]': {
        'Endpoint': {
          'Attributes': {       # Custom Endpoint Attributes Defined
            'EmailPreference_DailyDeals': ['Subscribed']
          }
        },
        'Events': {
          '[SomeUniqueEventId]': {
            'EventType': 'custom.preference_update',
            'Attributes': {       # Custom Event Attributes Defined
              'UpdatedPreference': 'EmailPreference_DailyDeals'
            },
            'Timestamp': datetime.datetime.fromtimestamp(time.time()).isoformat()
          }
        }
      }
    }
  }
)
```

## Pattern: Send Pre-Computed Lists of Users to Amazon Pinpoint

ISVs that are able to perform advanced segmentation can import into Amazon Pinpoint pre-computed lists of users for marketers to execute against directly.  Amazon Pinpoint's segmentation capabilities are flexible and allow for imported static lists of endpoints from external sources.  This allows targeting to take place by an ISV's segmentation engine while using Amazon Pinpoint's channel execution and orchestration capabilities.  

Similar to the [Send Users and Endpoints to Amazon Pinpoint](#Pattern-Send-Users-and-Endpoints-to-Amazon-Pinpoint) pattern above, ISVs can use the [CreateImportJob API](https://docs.aws.amazon.com/pinpoint/latest/apireference/apps-application-id-events.html#apps-application-id-eventspost) to import static CSV or JSON files containing lists of segments. If immediate execution is required, architectures can be set up, as seen in [this reference architecture](https://docs.aws.amazon.com/pinpoint/latest/apireference/apps-application-id-events.html#apps-application-id-eventspost), to detect new import files, begin an import job, and then immediately execute a send.

##### Example: Importing a named segment from CSV file generated by an external segmentation service
```python
import boto3
client = boto3.client('pinpoint')

response = client.create_import_job(
    ApplicationId='[PinpointProjectId]',
    ImportJobRequest={
        'DefineSegment': True,  # Allows for definition of a named segment in Amazon Pinpoint
        'Format': 'CSV',
        'RoleArn': 'arn:aws:iam::account-id:role/role-name-with-path',
        'S3Url': 's3://bucket-name/folder-name/file-name.csv',
        'SegmentName': 'Segment 123 from CDP'
    }
)
```

## Pattern: Get all Endpoints for a User

Amazon Pinpoint customers can allow their end-users to update their preferences and attributes directly through frameworks like [AWS Amplify](https://docs.amplify.aws/) and solutions like the [Amazon Pinpoint Preference Center](https://aws.amazon.com/solutions/implementations/amazon-pinpoint-preference-center/).  It can therefore be necessary for ISVs to retrieve the data in Amazon Pinpoint to update the central other systems.

ISVs can use the [GetUserEndpoints API](https://docs.aws.amazon.com/pinpoint/latest/apireference/apps-application-id-users-user-id.html#apps-application-id-users-user-idget) to retrieve all known endpoints, addresses, and attributes known to Amazon Pinpoint.

##### Example: Retrieving all endpoints for a user
```python
import boto3
client = boto3.client('pinpoint')

response = client.get_user_endpoints(
    ApplicationId='[PinpointProjectId]',
    UserId='[UserIdThatConnectsEndpoints]'
)
```

## Pattern: Consume Amazon Pinpoint's Engagement Events

Amazon Pinpoint generates engagement events as [campaigns](https://docs.aws.amazon.com/pinpoint/latest/developerguide/event-streams-data-campaign.html) and [journeys](https://docs.aws.amazon.com/pinpoint/latest/developerguide/event-streams-data-journey.html) execute, and [email](https://docs.aws.amazon.com/pinpoint/latest/developerguide/event-streams-data-email.html) and [sms](https://docs.aws.amazon.com/pinpoint/latest/developerguide/event-streams-data-sms.html) message are delivered and interacted with.  All of these events can be exported out of Amazon Pinpoint in real-time to Amazon Kinesis by using the [event stream](https://docs.aws.amazon.com/pinpoint/latest/developerguide/event-streams.html). In addition to containing events generated by Amazon Pinpoint, the event stream will also include any custom events fed to the [PutEvents API](https://docs.aws.amazon.com/pinpoint/latest/apireference/apps-application-id-events.html#apps-application-id-eventspost)].  This allows for one central location for all user events to be collected and exported.

ISVs can consume the event stream by connecting to Amazon Kinesis to receive the events in real time.  Customers can configure the event stream to write all events into an [Amazon Kinesis Data Stream](https://docs.aws.amazon.com/streams/latest/dev/introduction.html) that ISVs can then read.  ISVs can use these events to become learn about end-user behavior, such as topics that they engage with more frequently, as well as important data, such as email addresses that do not exist and cause hard bounces.

##### Example: Reading the Event Stream from Amazon Kinesis Data Streams looking for an email hard bounce event
```python
import boto3
import json
import base64
client = boto3.client('pinpoint')

def lambda_handler(event, context):
  for record in event['Records']:
    payload = base64.b64decode(record["kinesis"]["data"])
    pinpoint_event = json.loads(payload)
    if pinpoint_event['event_type'] == '_email.hardbounce':
      endpoint_id = pinpoint_event['client']['client_id']
      # TODO - call the CDP to inform that endpoint_id's email address is invalid

```

A full list of events available out of the box by Amazon Pinpoint's event stream can be found [here](https://docs.aws.amazon.com/pinpoint/latest/developerguide/event-streams.html).



## Pattern: Build Amazon Pinpoint Custom Channels

Amazon Pinpoint has four channels available natively out of the box including: Email, SMS, Push for Mobile Devices, and Voice.  Additionally, Amazon Pinpoint ISVs can create their own [custom channels](https://docs.aws.amazon.com/pinpoint/latest/developerguide/channels-custom.html) allowing marketers to send messages across any "channel" using any service that can be reached by an AWS Lambda function.  

A custom channel could be built to send messages to users on social media platforms by calling APIs on popular services like Facebook, Twitter, and WhatsApp.  Email and SMS messages could be sent using alternative email and SMS providers by calling other platform APIs. A custom channel could simply just write a message into a message database, such as DynamoDB, that is later retrieved by a web or mobile app on page load.

Custom channels can be used in creative ways that do not involve messaging, such as calling an API to update a customer record to keep track of what path in a Journey the user went down after a random split.  A custom channel could be built to allow marketers to assign offers by retrieving a user-specific offer code from a provisioning service, writing it back to the customer profile to be later used in an email.

Combining all of this, a collection of custom channels could enable a marketer to randomly assign users an offer, write that back to the customer record on file, then ensure that every touch point the user has will consistently show the offer details including email, sms, Facebook, and the website.

##### Example: Executing a Custom Channel AWS Lambda function
```python
import json

def lambda_handler(event, context):
  for endpoint_id in event['Endpoints']:

    endpoint_profile = event['Endpoints']['endpoint_id']
    user_id = endpoint_profile['User']['UserId']

    # TODO - call CDP to inform that User with user_id received a 20% off offer
    # or TODO - Call Facebook to write a message
    # or TODO - write into a database the details of the journey path they are on
```

##### Open Source Channels

There are a few open-source examples of custom channels found in GitHub:
* [WhatsApp](https://github.com/aws-samples/amazon-pinpoint-whatsapp-channel)
* [IFTTT](https://github.com/aws-samples/amazon-pinpoint-ifttt-channel)
* [Salesforce CRM](https://github.com/aws-samples/amazon-pinpoint-salesforce-channel)
* [Slack](https://github.com/aws-samples/amazon-pinpoint-slack-channel)
* [Twitter](https://github.com/aws-samples/amazon-pinpoint-twitter-channel)
* [Amazon Connect](https://github.com/aws-samples/amazon-pinpoint-connect-channel)

## Pattern: Build Amazon Pinpoint SMS Two-way Chat

Amazon Pinpoint can be used to send and receive SMS text messages.  After provisioning an SMS sending identity that supports two-way SMS, Amazon Pinpoint can be configured for [Two-Way SMS](https://docs.aws.amazon.com/pinpoint/latest/userguide/channels-sms-two-way.html) and will route all messages sent from end-users into an Amazon SNS Topic. Using this configuration, and other AWS Services, many two-way SMS architectures have been built.  Using Amazon Lex, customers can build an [SMS Chatbot](https://aws.amazon.com/blogs/messaging-and-targeting/create-an-sms-chatbox-with-amazon-pinpoint-and-lex/).  Using Amazon Connect, customers can [connect SMS to Amazon Connect's chat](https://github.com/Ryanjlowe/amazon-pinpoint-sms-connect-chat).

ISVs can configure Amazon Pinpoint to write all incoming SMS messages from end-users into an Amazon SNS Topic that they have read access to. From there, an ISV can do process the incoming messages, formulate conversations, and send replies using Amazon Pinpoint's [Send Message API](https://docs.aws.amazon.com/goto/WebAPI/pinpoint-2016-12-01/SendMessages).

##### Example: Processing an incoming message from Amazon Pinpoint's Two-Way SMS via Amazon SNS
```python
import json
import boto3

def lambda_handler(event, context):
  for record in event['Records']:
    messagejson = record['Sns']['Message']
    message = json.loads(messagejson)
    customer_txt_message = message['messageBody']
    customer_num = message['originationNumber']
    origination_id = message['destinationNumber']

    # TODO process the customer_txt_message from user with number  customer_num
```


## Pattern: Experimental Campaign Filters

Amazon Pinpoint's campaign targeting capabilities can be extended to perform [just-in-time filtering through the use of the beta feature, campaign hooks](https://docs.aws.amazon.com/pinpoint/latest/developerguide/segments-dynamic.html). When a campaign is configured to use campaign hooks, Amazon Pinpoint's standard segmentation is used, but at time of send and before the messages are uniquely rendered, Amazon Pinpoint will pass all of the targeted endpoints, in batches of 100, to an AWS Lambda function.  This function can then mutate the list of endpoints to either reduce the list or augment the attributes to be used for rendering.  NOTE: As of this time, this functionality is only available to Amazon Pinpoint campaigns, not journeys.

ISVs can use this functionality to perform a variety of interesting integrations.  ISVs can build AWS Lambda functions to perform filtering of endpoints for audit checks or enforcement of message frequency capping rules.  ISVs can also build integrations to [fetch data from other systems](https://github.com/aws-samples/digital-user-engagement-reference-architectures#send-time-amazon-pinpoint-campaign-attributes) to be made available in message rendering.  ISVs could even build an AWS Lambda function [fetch and render full templates from external systems](https://github.com/aws-samples/digital-user-engagement-reference-architectures#external-amazon-pinpoint-campaign-templates) to be used in the message send.

##### Example: Campaign Hook AWS Lambda function to retrieve data to used for rendering
```python
import json
import mock_offer_system

def lambda_handler(event, context):
  for endpointId,endpoint in event['Endpoints'].items():
    # TODO - call a webservice, look up a value from a database, call a CRM API to retrieve an offer for this endpoint
    offer_data = mock_offer_system.retrieve_offer_for_endpoint(endpointId)
    # add offer data to endpoint attributes so it can be used for message rendering
    endpoint['Attributes']['Offer'] = [offer_data]

  # return mutated endpoints
  return event['Endpoints']
```

## Other Amazon Pinpoint Resources
* [Amazon Pinpoint Home](https://aws.amazon.com/pinpoint/)
* [AWS Digital User Engagement Videos and Recorded Webinars](https://awsdue.tv/)
* [Digital User Engagement Reference Architectures](https://github.com/aws-samples/digital-user-engagement-reference-architectures)
* [AWS Solutions](https://aws.amazon.com/solutions/implementations/?solutions-all.sort-by=item.additionalFields.sortDate&solutions-all.sort-order=desc&solutions-all.q=pinpoint&solutions-all.q_operator=AND)
* [Amazon Pinpoint Documentation](https://docs.aws.amazon.com/pinpoint/)
