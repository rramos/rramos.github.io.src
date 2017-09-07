---
title: How to Create a Google Analytics Custom Dimension
date: 2017-09-07 20:37:23
tags:
    - Google Analytics
    - Analytics
    - Metrics
    - Dimensions
---

In order to add a custom dimension in Google Analytics follow this steps:

1. Click **admin** and navigate to the property you which to add a custom dimension

{% asset_img "Custom_Dimension.png" "Custom Dimension (step 1)" %}

2. Click new **Custom Dimension**

3. Give it a **Name** and a **Scope** and you'l get the javascript to add on your website

{% asset_img "GA_CustomDimension_js.png" "Custom Dimension (step 2)" %}

4. After you should modify your tracking code adding for example

```
ga('send', 'pageview', {
  'dimension1':  'My Custom Dimension'
});
```

## When we should use this ?

Let's say your website does some sort of user classification, you can create a CustomDimension ex: `UserCategory` and send enrich the data you track.
