# Amazon Pinpoint Key Concepts

Amazon Pinpoint is a multi-channel digital engagement service. It is a part of the Customer Engagement suite of services, enabling customers to send both campaign and transactional messages across email, SMS, push notification, voice, and custom channels.

## Key Concepts

* **Project** - Base structure inside of Pinpoint which is a collection of settings, end-user information, segments, campaigns, journeys, and analytics.
* **Endpoint** - An addressable destination where you can send a message.  An endpoint may be an email address, sms phone number, or a push token for a mobile device.  An endpoint is composed of an address and attributes.  Attributes can be used for message personalization and for filtering in segmentation.
* **User** - Endpoints can be tied together by specifying a common UserId on each endpoint.  User attributes can also be specified that will be synced across all of the User's endpoints.
* **Segment** - Collection of endpoints that have some meaningful connection.  There are two types of Segments:
  * **Import** - A CSV or JSON file that has been imported into Amazon Pinpoint to create a static immutable list.
  * **Dynamic** - Set of filters applied to endpoint attributes to create filtered lists evaluated every they are used.
* **Campaigns** - Allow you to send a message you specify, to a segment you specify at a time you specify.  It has features such as A/B testing, custom channel support, recurring schedules, and the ability to trigger messaging in response to custom events that are fed to Pinpoint’s Events API.
* **Journeys** - Multi-step experience feature that you create for your customers across multiple channels all at the same time.  It utilizes a drag and drop, no-code interface that is ideal for stakeholders who want to utilize conditional logic and a/b testing for journey-based engagement.

## References
* [What is Amazon Pinpoint?](https://docs.aws.amazon.com/pinpoint/latest/developerguide/welcome.html)
* [Adding endpoints to Amazon Pinpoint](https://docs.aws.amazon.com/pinpoint/latest/developerguide/audience-define-endpoints.html)
* [Associating users with Amazon Pinpoint endpoints](https://docs.aws.amazon.com/pinpoint/latest/developerguide/audience-define-user.html)
* [Creating segments](https://docs.aws.amazon.com/pinpoint/latest/developerguide/segments.html)
* [Creating campaigns](https://docs.aws.amazon.com/pinpoint/latest/developerguide/campaigns.html)
* [Amazon Pinpoint journeys](https://docs.aws.amazon.com/pinpoint/latest/userguide/journeys.html)
