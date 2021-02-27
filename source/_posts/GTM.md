---
title: Google Tag Manager
tags:
  - Google Tag Manager
  - Google Analytics
  - GA
  - GA4
---

In this article it will be provided a quick walkthrough on Google Tag Manager.

## What is Google Tag Manager ?

Google Tag Manager is a tag management system (TMS) that allows you to quickly and easily update measurement codes and related code fragments collectively known as tags on your website or mobile app. Once the small segment of Tag Manager code has been added to your project, you can safely and easily deploy analytics and measurement tag configurations from a web-based user interface.

### What is a Tag ?

Tags are segments of code provided by analytics, marketing, and support vendors to help you integrate their products into your websites or mobile apps. With Google Tag Manager, you no longer need to add these tags directly to your projects. Instead, you configure and publish tags and how they fire from within the Tag Manager user interface.

When we speak of tags in an HTML context, we refer to tags such as `<body>`, `<p>`, `<li>`, `<blockquote>`, and so on. When we refer to tags used in the analytics and marketing industry, we refer to code that an organization provides to install the desired product or functionality on your website or mobile app.

Tag Manager natively supports many [Google and 3rd party tag](https://support.google.com/tagmanager/answer/6106924) configurations. [Custom tags](https://support.google.com/tagmanager/answer/6107167) may be used to implement tags that are not yet supported by Tag Manager's native templates.

## How it works ?

* A snippet of code (container) is required to be installed on each page of your website like it happens for instance in Google Analytics.
* After that one needs to setup the triggers that would fire the tags using the tagmanager UI.
* After publishing those tags the events are triggered based on the defined rules in a asynchronous way.

> **NOTE:** You need one tag manager account per company or organization, but if you are tracking for several companies, you can connect to multiple tag manager accounts to a single user's Google account.
> Each Tag Manager account has at least one container.  A container includes the tags and “triggers” that determine when those tags should fire or collect data. Typically, you’ll have one container for each website domain, though you can use a single container for cross-domain tracking

### How to Setup ?

In order to setup GTM the following steps are required:

1. **Create an account**, or use an existing account, at [tagmanager.google.com](tagmanager.google.co). (A new container is created by default, and you can create additional containers within each account.)
2. **Install the container** in your website or mobile app.
    * For web and AMP pages: Add the container snippet as instructed by Tag Manager, and remove any existing tags.
    * For mobile apps: Use the Firebase SDK for Android or iOS.
3. **Add and publish your tags**.

> **NOTE:** A Tag Manager account represents the topmost level of organization. Typically, only one account is needed per company. A Tag Manager account contains one or more containers, and there are specific container types that may be used for websites, AMP pages, Android apps, and iOS apps.

#### **Install the snippet**

* Place the `<script>` code snippet in the `<head>` of your web page HTML output, preferably as close to the opening `<head>` tag as possible, but below any dataLayer declarations.

* Place the `<noscript>` code snippet immediately after the `<body>` tag in your HTML output.

> **NOTE:** To ensure that tags do not fire twice, remove from your code any hardcoded tags that have been migrated to your Tag Manager container.

## Why should one use Google Tag Manager ?

* Tag manager allows to simplify the management of tags and reduce the dependency from engineering teams. Acts as a bridge between Marketers and developers.
* Allow to have a collaborative space for marketing teams to test and source control their changes, improving the workflow to have this changes deployed.
* Improves page loads as the tags are fired asynchronously.

## How to use Google Tag Manager ?

### Manage Tags

Tags are fired based on triggers and these make use of:

* Variable
* Operators
* Values

#### Variables

* **Built-in Variables**: ex: `{{page path}}` or `{{click id}}`
* **User defined variables**: ex: `{{purchaseComplete}}`

#### Operators

* Operators define the relation with variable and values: ex: *contains, equals*

#### Values

* Values like numbers or strings: ex: 1, 2, 3 or a, b , c

When the user visit the site the container snippet will trigger tags based on firing instructions which communicates to systems such as Google Analytics.

## DataLayer 

The Data Layer in Tag Manager is a Javascript object that holds data such as custom event information or variables passed from your website. Information in the Data Layer is structured into key-value pairs, which may be passed to third-party applications like Analytics or used as a trigger to determine when tags should fire.

There are two ways to populate the Data Layer with these key-value pairs.

 1. Pre-populating values in the data layer
 2. Javascript push into the Data Layer
 
> **NOTE**: It’s important to note that Data Layer variables don’t persist across pages automatically. If you wish to pass a value to the Data Layer on additional pages, you’ll need to write custom code to do so.

## Definitions

* **KPI** - Key Performance Indicator (KPI) Definition
A Key Performance Indicator is a measurable value that demonstrates how effectively a company is achieving key business objectives. Organizations use KPIs at multiple levels to evaluate their success at reaching targets.

## References

* <https://marketingplatform.google.com/about/tag-manager/>
* <https://developers.google.com/tag-manager>
* <https://support.google.com/tagmanager/answer/6102821?hl=en>