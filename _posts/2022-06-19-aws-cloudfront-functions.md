---
layout: post
title: Hardening AWS hosted websites using AWS CloudFront functions 
date: 2022-06-19
---

This is a short guide to harden AWS hosted websites injecting security headers in their HTTP response using AWS CloudFront functions.

*Do you have AWS CLI version 2 installed and configured? If don't, please follow this <a href="https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html" title="Installing or updating the latest version of the AWS CLI" target="_blank">guide</a>.*

**Steps**

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