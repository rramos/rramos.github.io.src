---
title: Google Analytics - UA vs GA4
tags:
   - GA
   - Google Analytics
   - GA4
---

## Google Analytics quick History

In the October of 2020, Google rolled out Google Analytics 4 (GA4), the latest iteration of the Google Analytics platform. GA4 replaces Universal Analytics (UA) as the default for digital analytics measurement in GA.

The objective of this article is to consolidate the most significative changes, and required operations to migrate to the new data model.

## Differences in UA and GA4 data models

### Events and Hit types

The most significative change from UA to GA4 in my opinion is the fact that all interactions are now reported as events.

In UA an event has a Category, Action and Label for that specific hit type. In GA4 all hits are events there is no distinction on the hit types

| Hit type in (UA)        | Measurement in GA4  |
|-------------------------|---------------------|
| Page View               | Event               |
| Event                   | Event               |
| Social                  | Event               |
| Transaction/e-commerce  | Event               |
| User timing             | Event               |
| Exception               | Event               |
| App/screen view         | Event               |

----

GA4 events fall into four categories:

* Automatically collected events
* Enhanced measurements events
* Recommended events
* Custom events

#### **Automatically collected events**

This events are logged as long you have the base code for GA4 snippet on your site either via gtag.js or Google Tag Manager. No code required

#### **Enhanced Measurements**

Enhanced measurements events are events that are automatically logged no code required and can be enable/disabled on the properties. also they don't count for the named events limit that we will check next. They allow you to measure interactions with your content.

#### **Recommended Events**

This events have predefined names and parameters and are recommend for specific business use cases. This events allow you to have more detail on your reports and enables the benefit from latest features but require you to add code as they are not logged automatically.

They are recommended for:

* Retail and e-commerce
* Jobs/Education, Local Deals, Real Estate
* Travel
* Games

#### **Custom Events**

Custom events are implemented by yourself. You should use as a last resource and identify if the previous ones provide the data you require.

### Pageviews

Page view in UA translate to the `page_view` event . This event is trigger automatically using the gtag.js snippet or by Google Tag manager. For Mobile apps the analog to a pageview a *screenview* is mapped to a `screen_view` event

### Sessions

Sessions is a group of interactions with your website that takes place within a given time frame. In UA sessions are typically defined as having ended once there has been a 30m period of inactivity or another reset event. In GA4 session metrics are derived from session_start event and the last event in the session.

* **Active user calculation**: User activity is now detected automatically in GA4 in contrast with UA. Due to the fact that the ID can span to multiple devices it's normal that we find mismatch when comparing both UA and GA4 data.
* **Session counting**: One important change is that a new Campaign does not reset the session in GA4 has occurs in UA. Also GA4 processes the sessions with arrives up to 72 hours late while in UA the processing takes place getting data if they arrive 4 hours of close of the preceding day. Another important feature is that the iOS SDK uploads metrics automatically when apps are background, this can also result in different values if compared with UA data.

### Custom Metrics and Dimensions

Custom Metrics and Dimensions have their own scope. When changing to GA4 the following mapping should be applied.

| Scope in UA             | GA4 property          |
|-------------------------|-----------------------|
| Hit-scoped              | Events or parameters  |
| User-scoped             | User properties       |
| Session-scoped          | N/A                   |
| Product-scoped          | E-commerce parameters |

Remember that events, event parameters and user properties are subject to limits.

### Content Grouping

This is also big change as in UA you could group content into a logical structure, and then compare metrics by group. (ex: Man/Shirts). In GA4 there is a predefined event `content_group` that populates data into a "Content Group" Dimension

### Parameters

In GA4 you can send parameters with each event. Parameters are additional pieces of information that can further specify the action the user took, or add more context.Each event can log up to 25 parameters.

### User property

User properties are attributes about the users who interact with your app or website. They are used to describe segments of your user base.

## Data collection settings that can be migrated

The following data collection settings migrate in one-to-one from UA to GA4 as long as you use the gtag.js snippet or Google tag Manager.

* cookie customization
* ads personalization

## Data collection settings with no equivalent in GA4

* control over IP anonymization - Enabled by default in GA4 properties
* get clientID - Not currently available
* custom task - Not available in GA4
* timing - Not available in GA4

## Limits

Collections and configurations limits at the date:

* Distinctly named events **500** per app instance. (Can not be deleted if reaching limit)
* Length of event names is **40** characters
* Limit of **25** parameters per event
* Parameters value have the length limit of **100** characters
* **25** user properties that can be added
* length of user property names is **24** charaters
* length of user property values is **36** characters
* **100** audiences can be created by property
* **30** convertions can be created by property
* Data retention up to **14** months

For a complete list check the following [article](https://support.google.com/analytics/answer/9267744)

## Comparing report data in UA and GA4

As refereed there are several changes that may impact your report when shifting from UA to GA4. Google recommends that one creates a new tracking ID and keep both tracking options working simultaneously, preferably manged in Google Tag manager.

Make sure of the following:

* Your tracking ID from UA and GA4 are both collecting the same web pages
* Both properties have equivalent tag implementations 
* All tags are firing successfully
* Both properties use the same time zone
* Make sure to compare an unfiltered view in UA with a single web data stream in GA4
* Both properties have been collecting data for at least 30m in order to compare on the realtime report


Cheers,
RR