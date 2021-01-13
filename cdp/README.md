# Customer Data Platform (CDP) ISVs

Amazon Pinpoint is not a CDP, but it does work very well when connected to a CDP. A typical CDP is the authoritative source of truth for all users and their respective addresses. A CDP will get all of this information by connecting to a variety of different data sources in order to build a true picture of the end-user, all of their attributes, their behavioral data, and communication and channel preferences.  This information becomes invaluable for Amazon Pinpoint to effectively filter, personalize, and execute communications.

ISVs should think of Amazon Pinpoint as a downstream data store for users and their communication channels (endpoints) and preferences (attributes).  CDP ISVs would best integrate by syncing this data to Amazon Pinpoint in a consistent fashion so that the data in Amazon Pinpoint is relevant and consistent for marketing and other communications teams.  ISVs are encouraged to NOT build their CDP or other customer data store on top of Amazon Pinpoint in a way that makes Amazon Pinpoint the central source of truth for customer data.

CDPs have a variety of different ways that they can integrate with Amazon Pinpoint to the benefit of joint customers.  CDPs can:

### Update Amazon Pinpoint when a new user is created in the CDP with all known addresses

This example uses the [Send Users and Endpoints to Amazon Pinpoint](../../../#Pattern-Send-Users-and-Endpoints-to-Amazon-Pinpoint) pattern.

In the below example, we have configured the CDP to call Amazon Pinpoint whenever a new user is created and include all of the necessary endpoints used for communication.  We are sending User Attributes including first and last name.  We are also sending over communication preferences so that the marketing team has access to the user's preferences and can abide by them when creating campaigns and journeys in Amazon Pinpoint.

```python
import boto3
client = boto3.client('pinpoint')

response = client.update_endpoints_batch(
  ApplicationId='[PinpointProjectId]',
  EndpointBatchRequest={
    'Item': [
      {
        'Address': 'customer@example.com',
        'Attributes': {       # Custom Endpoint Attributes Defined
          'EmailPreference_Newsletter': ['Subscribed'],
          'EmailPreference_DailyDeals': ['Unsubscribed']
        },
        'ChannelType': 'EMAIL',
        'Id': '[UniqueEndpointID1ForUser]',
        'OptOut': 'NONE',
        'User': {
          'UserAttributes': {       # Custom User Attributes Defined
            'FirstName': [ 'John'],
            'LastName': [ 'Doe']
          },
          'UserId': '[UserIdThatConnectsEndpoints]'
        }
      }, {
        'Address': '+15555555555',
        'Attributes': {       # Custom Endpoint Attributes Defined
          'SMSPreference_Deals': ['Unsubscribed'],
        },
        'ChannelType': 'SMS',
        'Id': '[UniqueEndpointID2ForUser]',
        'OptOut': 'ALL',
        'User': {
          'UserAttributes': {       # Custom User Attributes Defined
            'FirstName': [ 'John'],
            'LastName': [ 'Doe']
          },
          'UserId': '[UserIdThatConnectsEndpoints]'
        }
      }, {...}
    ]
  }
)
```


### Send Real-time events from the CDP to Amazon Pinpoint

This example uses the [real-time events to Amazon Pinpoint](../../../#Pattern-Send-User-Events-to-Amazon-Pinpoint) pattern.

In the below example, we have configured the CDP to call Amazon Pinpoint whenever an attribute or piece of data changes for a user.  In the first example we are submitting a changed communication preference.  We can specify the event attribute `UpdatedPreference` to allow marketers to filter events to specific preference updates when creating campaigns or journeys.  The second example shows a purchase event.  Both examples will update the Endpoint with new data, but the events could also be used to trigger an Amazon Pinpoint Campaign or Journey in real-time.

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
        'Endpoint': { },
        'Events': {
          '[SomeUniqueEventId]': {
            'EventType': 'custom.purchase_event',
            'Attributes': {
              'Item': 'Hat'
            },
            'Timestamp': datetime.datetime.fromtimestamp(time.time()).isoformat()
          }
        }
      }
    }
  }
)
```

### Use Amazon Pinpoint to send messages across all messaging channels

This example uses the [Send a message using Amazon Pinpoint](../../../#Pattern-Send-a-message-using-Amazon-Pinpoint) pattern.

In the below example, the CDP is calling Amazon Pinpoint's Send Messages API to deliver an email message using a template.  In this way, the customer is able to centralize all message sending and perform better engagement reporting having visibility to all message touch points.

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


### Using the Create Import Job API to create a Segment from a list generated by the CDP.

This example uses the [Send pre-computed lists of users to Amazon Pinpoint](../../../#Pattern-Send-Pre-Computed-Lists-of-Users-to-Amazon-Pinpoint) pattern.

In the below example, we are generating a static imported segment in Amazon Pinpoint using a CSV file generated by performing our own segmentation logic in our CDP and exporting to Amazon S3.

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

### Listening to the Amazon Pinpoint Event Stream via Kinesis Data Streams to update the CDP

This example uses the [Consume Amazon Pinpoint's engagement events](../../../#Pattern-Consume-Amazon-Pinpoints-Engagement-Events) pattern.

In the below example, we are listing specifically for the `_email.hardbounce` event so that we can inform the CDP that the user email address is invalid.  This sample assumes that it is running inside of an AWS Lambda function with a configured event source of Kinesis Data Streams - AWS Lambda is not required for this use-case.

A full list of events that can be consumed to inform the CDP are listed [here](https://docs.aws.amazon.com/pinpoint/latest/developerguide/event-streams.html).

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


### Amazon Pinpoint Custom Channel to update CDP that an Offer was assigned

This example uses the [Build Amazon Pinpoint custom channels](../../../#Pattern-Build-Amazon-Pinpoint-Custom-Channels) pattern.

In the below example, we have built an [Amazon Pinpoint custom channel](https://docs.aws.amazon.com/pinpoint/latest/developerguide/channels-custom.html) that can be dropped into an Amazon Pinpoint journey.  This custom channel writes back to the CDP that a 20% off offer was provided to the user.  This custom channel could be used in an Amazon Pinpoint journey after a random split to assign a portion of the audience a 20% off offer.  In this way, the end-users who go down this path in the journey will receive an offer email, but since this information is written back to the CDP, other user touch points can be informed of the offer as well.  Such as validating an offer on checkout.

```python
import json

def lambda_handler(event, context):
  for endpoint_id in event['Endpoints']:
    user_id = event['Endpoints']['endpoint_id']['User']['UserId']
    # TODO - call CDP to inform that User with user_id received a 20% off offer

```
