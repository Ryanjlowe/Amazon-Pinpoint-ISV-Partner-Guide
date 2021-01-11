# Amazon Pinpoint ISV Partner Guide

This repository serves as a guide for how Independent Software Vendors (ISVs) and other integration partners can build integrations between their product/service and Amazon Pinpoint.  It showcases common integration patterns that Amazon Pinpoint can be integrated with sample use-cases and examples.

The below guide provides examples and patterns that an ISV could use to integrate their application with Amazon Pinpoint.  Use the links below to find pages specific to the ISVs listed:
* [Customer Data Platform (CDP) ISVs](cdp/)
* [Application Component ISVs](app/) - coming soon!
* [Custom Channel ISVs](channel/) - coming soon!
* [Identity Management ISVs](identity/) - coming soon!

## What is Amazon Pinpoint
Amazon Pinpoint is a multi-channel digital engagement service. It is a part of the Customer Engagement suite of services, enabling customers to send both campaign and transactional messages across email, SMS, push notification, voice, and custom channels.  For more details, and a quick guide to Pinpoint terms, see [Amazon Pinpoint Key Concepts](pinpoint_detail/README.md).

## Note on Below Integration Patterns

Below is a set of common integration patterns for Amazon Pinpoint.  This is not an exhaustive list and customers and partners [can use the APIs](https://docs.aws.amazon.com/pinpoint/latest/apireference/welcome.html) to build any kind of integration. Example source code in Python is shown using the [AWS Python Boto3 SDK](https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/pinpoint.html).  Integrations can choose to use one of the AWS SDKs or call the REST APIs directly.  No SDK is necessary when integrating with Amazon Pinpoint.

## Pattern: Send Users and Endpoints to Amazon Pinpoint
ISVs that manage users and user addresses can send this data to Amazon Pinpoint. This can be used to help build marketing campaign audiences, by providing new users and addresses as they are created, or by augmenting the current set of users and addresses in Amazon Pinpoint by enriching with new data attributes.  These attributes can then be used for message personalization, such as `First Name` or `Item Purchased`, or for filtering when creating dynamic segments to create specific audiences, such as `High Value Customer`, `Users Nearby` or `Newsletter Subscription Status`.

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
        'DefineSegment': True,
        'Format': 'CSV',
        'RoleArn': 'arn:aws:iam::account-id:role/role-name-with-path',
        'S3Url': 's3://bucket-name/folder-name/file-name.csv',
        'SegmentName': 'Segment 123 from CDP'
    }
)
```


## Pattern: Consume Amazon Pinpoint's Engagement Events

##### Example:
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


## Pattern: Export Amazon Pinpoint's Segments
High-level... technical details [here](export/README.md)

## Pattern: Build Amazon Pinpoint Custom Channels

##### Example:
```python
import json

def lambda_handler(event, context):
  for endpoint_id in event['Endpoints']:
    user_id = event['Endpoints']['endpoint_id']['User']['UserId']
    # TODO - call CDP to inform that User with user_id received a 20% off offer

```

## Pattern: Build Amazon Pinpoint SMS Two-way Chat
High-level... technical details [here](two_way_sms/README.md)
