# Custom Channel ISVs

Amazon Pinpoint supports channel support includes Email, SMS, Push, and Voice.  Amazon Pinpoint can also be configured to send messages through new [Custom Channels](https://docs.aws.amazon.com/pinpoint/latest/developerguide/channels-custom.html).  With Custom Channels, you can configure Amazon Pinpoint to send messages or notifications through any service that has an API, including ISV built messaging systems. The custom channel can used as the destination for a campaign or included as an activity in a journey.  Custom channels are executed with a web-hook or an AWS Lambda function where the context of the Journey/Campaign, along with the list of targeted endpoints, are sent.

ISVs are encourage to report status events, such as success and failure, back to Amazon Pinpoint for reporting engagements across all channels in the event stream.

### Calling a Third-Party API from an Amazon Pinpoint Custom Channel to send a message to a user

This example uses the [Build Amazon Pinpoint SMS Two-way Chat](../../../#Pattern-Build-Amazon-Pinpoint-SMS-Two-way-Chat) pattern.

In the below example, we are creating an AWS Lambda function that will trigger an outgoing message using the ISV's own message system.  

```python
import isv_api

def lambda_handler(event, context):
  for endpoint_id in event['Endpoints']:

    endpoint_profile = event['Endpoints']['endpoint_id']
    user_id = endpoint_profile['User']['UserId']
    address = endpoint_profile['Address']

    message = 'This is a message to deliver to the user.'
    isv_api.send_message(address, message)

```

### Reporting Engagement Events to Amazon Pinpoint for Reporting

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
        'Endpoint': {},
        'Events': {
          '[SomeUniqueEventId1]': {
            'EventType': 'isv_channel.delivery',
            'Attributes': {       # Custom Event Attributes Defined
              'Status': 'Message was successfully delivered'
            },
            'Timestamp': datetime.datetime.fromtimestamp(time.time()).isoformat()
          },
          '[SomeUniqueEventId2]': {
            'EventType': 'isv_channel.open',
            'Attributes': {       # Custom Event Attributes Defined
              'Status': 'Message was opened by end user'
            },
            'Timestamp': datetime.datetime.fromtimestamp(time.time()).isoformat()
          }
        }
      }
    }
  }
)
```
