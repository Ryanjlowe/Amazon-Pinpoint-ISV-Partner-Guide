# Application Development Framework ISVs

One of Amazon Pinpoint's original use-cases was to be used in conjunction with the Application Development Framework [AWS Amplify](https://docs.amplify.aws/).  AWS Amplify is a set of tools and services that can be used together or on their own to help front-end web and mobile developers build applications using AWS services.  Specifically, AWS Amplify uses Amazon Pinpoint to register users and endpoints, stream app engagement events for reporting and to trigger campaigns, and trigger and receive push notifications.

Application Development Frameworks could create framework modules for Amazon Pinpoint.  Framework users would create an Amazon Pinpoint project, enable the different channels, then provide the framework credentials to call the Amazon Pinpoint APIs.

AWS Amplify, however, simply uses the exact same set of public APIs when interacting with Amazon Pinpoint.  This means any application development framework could utilize Amazon Pinpoint in the same way.  Below are a few different ways that Application Development Framework ISVs could integrate with Amazon Pinpoint:

### Register Users and Endpoints during account creation
This example uses the [Send Users and Endpoints to Amazon Pinpoint](../../../#Pattern-Send-Users-and-Endpoints-to-Amazon-Pinpoint) pattern.

In the below example, we can configure our application integration to call Amazon Pinpoint whenever a new user is created and include all of the necessary endpoints, endpoint and user attributes, and communication preferences. We are sending User Attributes including first and last name.  We are also sending over communication preferences so that the marketing team has access to the user's preferences and can abide by them when creating campaigns and journeys in Amazon Pinpoint.

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

### Submit Real-time events for reporting and campaign triggers

This example uses the [real-time events to Amazon Pinpoint](../../../#Pattern-Send-User-Events-to-Amazon-Pinpoint) pattern.

In the below example, we can configure our application integration to call Amazon Pinpoint whenever a state change occurs that we want to track. Here, our imaginary application development framework tracks user's geo-location in the mobile app.  This is then used to alert Amazon Pinpoint to proximity events of store locations.  The sample code below submits a custom event to Amazon Pinpoint to notify that the current user is now in proximity to a store coded with number `12345`.  This could be then used to trigger a Push notification or SMS to the user to entice them to visit the store location with an offer.


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
            'EventType': 'custom.proximity_event',
            'Attributes': {
              'StoreNumber': '12345'
            },
            'Timestamp': datetime.datetime.fromtimestamp(time.time()).isoformat()
          }
        }
      }
    }
  }
)
```

### Trigger channel specific messaging

This example uses the [Send a message using Amazon Pinpoint](../../../#Pattern-Send-a-message-using-Amazon-Pinpoint) pattern.

In the below example, our application framework sends the end-user a one-time-password (`123456`) to verify the user's phone number.

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
