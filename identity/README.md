# Identity Management ISVs

Amazon Pinpoint's events and analytics capabilities work very well with end-user identity management providers. While Amazon Pinpoint does work out of the box with [Amazon Cognito to authenticate and authorize end-users to applications](https://docs.aws.amazon.com/cognito/latest/developerguide/cognito-user-pools-pinpoint-integration.html), Amazon Pinpoint is not locked down to only Amazon Cognito.  In fact, Amazon Cognito uses the same Amazon Pinpoint APIs that ISVs can use to build very similar integrations for the same use-cases.

The following [Amazon Pinpoint app events](https://docs.aws.amazon.com/pinpoint/latest/developerguide/event-streams-data-app.html) can be submitted by ISVs By matching the `EventType`'s to the below list that Amazon Cognito submits to Amazon Pinpoint in the [PutEvents API](https://docs.aws.amazon.com/pinpoint/latest/apireference/apps-application-id-events.html#apps-application-id-eventspost). These events would then be available to see on [Amazon Pinpoint's analytics charts](https://docs.aws.amazon.com/pinpoint/latest/userguide/analytics.html) and be used to help segment by providing [usage data](https://docs.aws.amazon.com/pinpoint/latest/userguide/analytics-usage.html).

* _session.start
* _session.stop
* _userauth.sign_in
* _userauth.sign_up
* _userauth.auth_fail
* _session.pause
* _session.resume

Below are examples of integrations that identity management ISVs can use to integrate with Amazon Pinpoint.

### Send to Amazon Pinpoint User Authentication and Session Events

##### Example: User Sign Up Event (_userauth.sign_up) that will also create a new Endpoint
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
          'Address': 'customer@example.com',
          'Attributes': {       # Custom Endpoint Attributes Defined
            'EmailPreference_Newsletter': ['Subscribed'],
            'EmailPreference_DailyDeals': ['Unsubscribed']
          },
          'ChannelType': 'EMAIL',
          'OptOut': 'NONE',
          'User': {
            'UserAttributes': {       # Custom User Attributes Defined
              'FirstName': [ 'John'],
              'LastName': [ 'Doe']
            },
            'UserId': '[UserIdThatConnectsEndpoints]'
          },
        },
        'Events': {
          '[SomeUniqueEventId]': {
            'EventType': '_userauth.sign_up',
            'Timestamp': datetime.datetime.fromtimestamp(time.time()).isoformat()
          }
        }
      }
    }
  }
)
```

##### Example: Batch of a User Sign In event (_userauth.sign_in) and a Session Start event (_session.start) while passing in Device Demographic data of the user's device
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
          'Demographic': {
            'AppVersion': '1.0',
            'Locale': 'en_US',
            'Make': 'generic web browser',
            'Model': 'Unknown',
            'ModelVersion': 'Unknown',
            'Platform': 'android',
            'PlatformVersion': '10.10',
            'Timezone': 'America/Los_Angeles'
          },
          'Location': {
            'City': 'Los Angeles',
            'Country': 'US',
            'Latitude': 123.0,
            'Longitude': 123.0,
            'PostalCode': '90210',
            'Region': 'California'
          },
        },
        'Events': {
          '[SomeUniqueEventId1]': {
            'EventType': '_userauth.sign_in',
            'Timestamp': datetime.datetime.fromtimestamp(time.time()).isoformat()
          },
          '[SomeUniqueEventId2]': {
            'EventType': '_session.start',
            'Timestamp': datetime.datetime.fromtimestamp(time.time()).isoformat()
          }
        }
      }
    }
  }
)
```
