---
layout: post
title: Hardening AWS hosted websites' security using AWS CloudFront functions 
date: 2022-06-19
---

This is a short guide to harden AWS hosted websites injecting security headers in their HTTP response using AWS CloudFront functions.

*Have you got AWS CLI version 2 installed and configured? If you haven't, please follow this <a href="https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html" title="Installing or updating the latest version of the AWS CLI" target="_blank">guide</a> and get it.*

## Build

1. Create a text file named *security-headers.js* with this code.
```javascript
function handler(event) {
    var response = event.response;
    response.headers["strict-transport-security"] = {
        value: "max-age=15768000; includeSubDomains"
    };
    response.headers["x-frame-options"] = {
        value: "deny"
    };
    response.headers["x-xss-protection"] = {
        value: "1; mode=block"
    };
    response.headers["permissions-policy"] = {
        value: "accelerometer=(), ambient-light-sensor=(), autoplay=(), battery=(), camera=(), cross-origin-isolated=(), display-capture=(), document-domain=(), encrypted-media=(), execution-while-not-rendered=(), execution-while-out-of-viewport=(), fullscreen=(), geolocation=(), gyroscope=(), keyboard-map=(), magnetometer=(), microphone=(), midi=(), navigation-override=(), payment=(), picture-in-picture=(), publickey-credentials-get=(), screen-wake-lock=(), sync-xhr=(), usb=(), web-share=(), xr-spatial-tracking=()"
    };
    response.headers["x-content-type-options"] = {
        value: "nosniff"
    };
    response.headers["content-security-policy"] = {
        value: "default-src 'none';style-src 'self';script-src 'self';img-src 'self';"
    };
    response.headers["referrer-policy"] = {
        value: "no-referrer-when-downgrade"
    };
    return response;
}
```

2. Upload the file to AWS using *aws cloudfront create-function* like this:
```bash
aws cloudfront create-function --name security-headers --function-config Comment="Security headers",Runtime="cloudfront-js-1.0" --function-code fileb://security-headers.js
```
Jot down the *ETag* value returned on the *JSON* response.

## Test

1. Create a text file named *event-object.json* with this code.
```javascript
{
    "version":"1.0",
    "context":{
       "eventType":"viewer-response"
    },
    "viewer":{
       "ip":"1.2.3.4"
    },
    "request":{
       "method":"GET",
       "uri":"/index.html"
    },
    "response":{
       "statusDescription":"OK",
       "statusCode":200
    }
 }
```

2. Test the function before issuing this command replacing Exxxxxxxxx with the *ETag* you jotted down when you build the function.
```bash
aws cloudfront test-function --name security-headers --if-match Exxxxxxxxx --event-object fileb://event-object.json --stage DEVELOPMENT
```
Look for the field *FunctionOutput* on its response. You should see the injected headers in it if the function is correct.

## Publish

To publish the function, just run this:
```bash
aws cloudfront publish-function --name security-headers --if-match Exxxxxxxxx
```
where Exxxxxxxxx must be replaced with the *ETag* you jotted down when you build the function.

## Deploy

*I recommend to use jq for JSON data filtering. With jq you can get just a property from a long JSON response. You can get it from <a href="https://stedolan.github.io/jq/download/" title="https://stedolan.github.io/jq/download/" target="_blank">here</a>.*

An AWS CloudFront function is associated with a website including its ARN on the AWS CloudFront distribution configuration. To do so, we need to get these values:

**Distribution Id**
```bash
aws cloudfront list-distributions | jq -r ".DistributionList.Items[] | {Id,AliasICPRecordals}"
```

**ETag**
```bash
aws cloudfront describe-function --name security-headers | jq -r ".ETag"
```

**ARN**
```bash
aws cloudfront describe-function --name security-headers | jq -r ".FunctionSummary.FunctionMetadata.FunctionARN"
```
