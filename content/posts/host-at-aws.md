---
title: "Host at AWS"
description: "A simple guide to hosting your static website at AWS!"

date: 2019-04-28T18:29:34+02:00
draft: true

categories:
- Development
- AWS
tags:
- AWS
- douwe
- alex
- lol
---

# Host at AWS

#### Written by Douwe den Blanken and Alex Vinarskis

## Introduction

In this guide, we will explain how to setup a web hosting environment in AWS for a static website. As we did not find this process to be very intuitive ourselves, we decided to write this guide to enable more people to make use of all of the great static hosting possibilities AWS has to offer.

This guide assumes that you have already created an AWS account, which if you haven't, can do [here](https://portal.aws.amazon.com/billing/signup#/start). Also, the guide assumes that you have your domain parked with GoDaddy. This guide however is also applicable to other domain name providers.

We chose to write this guide in a very step-by-step manner as we quite enjoy using those guides ourselves.Our recommendation is to open this document on one side of your screen and open your browser on the other side. This works very well as this document **exactly** describes how to setup everything up.

If you follow the steps in this document you are able to launch your website within 40 minutes, including the time you have to wait for AWS to initiate everything.

## Part 1: Setup your domain

The first part of this guide will be on how to setup your domain name within AWS.  Below is a clear list of steps on how to do this:

1. Sign in to the AWS console and go to [Route 53](https://console.aws.amazon.com/route53/)
2. From the menu on the left side of the screen, select 'Hosted zones'
3. Hit the blue button 'Create Hosted Zone'
4. In the newly opened menu, enter your domain name (for example: `domain.com`) and make sure that the dropdown is on 'Public Hosted Zone'

The following steps will only be exactly applicable to domains bought at GoDaddy, although the same concept can be carried through to other domain name providers.

1. Open a new tab and log in to [GoDaddy](https://sso.godaddy.com/login?realm=idp&app=dcc&path=%2f)
2. Go to the ['My Products' page](https://account.godaddy.com/products/)
3. Under the dropdown 'Domains', find the domain which you want to setup hosting for and click on the 'DNS' button on the right
4. On this newly opened page, scroll down to 'Name Servers'
5. Hit the blue button below 'Name Servers' which says 'Edit'
6. In this dropdown, select the option 'Custom'
7. Go back to your AWS console tab
8. Copy the different domains in the 'Value' column of the Type 'NS' (without the trailing point) and paste them in the list of name servers on the GoDaddy page
9. If you don't have enough inputs to fill in all of your name servers, hit the 'Add Name Server' button
10. When you are done copying the contents, hit the 'Save' button.

Now that your domain's DNS is transfered to AWS, the real fun can begin.

## Part 2: Setup file hosting via S3

Now, we will create a place to actually store your website. In AWS, this place is called Simple Storage Service (S3). Follow the steps below to create a bucket, which will eventually hold all of your web files:

1. Open another tab and go the [S3 page](https://s3.console.aws.amazon.com/s3/)
2. Now, hit the blue button saying 'Create bucket'

A new window opens, in which you have select the options for your bucket. Again, follow the following steps closely:

1. In the 'Bucket name' field, fill your domain name in (for example: `domain.com`)
2. Select your region, preferably the one which is closest to you
3. Click 'Next'
4. Click 'Next' again
5. Uncheck the options 'Block new public policies' and 'Block public and cross-account access if bucket ' under '**Manage public bucket policies for this bucket**'
6. Click 'Next' and after this, click 'Create bucket'

Now that your bucket has been created, the setup can continue.

1. Click on your newly created bucket
2. Now, go to the 'Permissions' section
3. Click on the 'Bucket Policy' button
4. Copy the the text below into the newly opened text editor, while replacing `domain.com` by your own domain name:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "PublicReadGetObject",
            "Effect": "Allow",
            "Principal": "*",
            "Action": "s3:GetObject",
            "Resource": "arn:aws:s3:::domain.com/*"
        }
    ]
}
```

6. Ignore the 'This bucket has public access' and go to the 'Properties' section
7. Click on the 'Static website hosting' block
8. Select 'Use this bucket to host a website'
9. In the 'Index document' field, type `index.html`
10. Click on the 'Save' button

Now, upload a random file or, if you have one, your `index.html` file (while uploading, just hit next every time).

## Part 3: CloudFront initiation

Now that your files are ready to be served via S3, we will configure CloudFront to make sure that your website's user can be talked to over SSL:

1. Go to [AWS CloudFront](https://console.aws.amazon.com/cloudfront/home)
2. Click on the blue button 'Create Distribution'
3. Select the top 'Get Started Button', under the 'Web' heading
4. In the 'Origin Domain Name' field, select your the S3 bucket you just created
5. Scroll down to 'Default Cache Behavior Settings'
6. Next to **Viewer Protocol Policy** select 'Redirect HTTP to HTTPS'
7. Now, scroll further down to 'Distribution Settings'
8. Next to **Price Class** select 'Use Only U.S., Canada and Europe' or leave it as is if you want only the best performance (which obviously is costs more money)
9. In the **Alternate Domain Names (CNAMEs)** field type `www.domain.com`, press enter and type `domain.com` where 'domain.com' should obviously be replaced by your own domain name
10. Leave the rest of the settings as they are and scroll down, where you hit the 'Create Distribution' button

## Part 4: SSL certificate

Now while you can see that the status of creating the distribution is in 'In Progress', we are going to acquire an SSL certificate.

1. Go to [AWS Certificate Manager](https://console.aws.amazon.com/acm/home)
2. Make sure that your selected region is 'US East (N. Virgina)' as CloudFront only accepts certificates hosted here
3. Click on the blue button 'Request a certificate'
4. Click again on 'Request a certificate'
5. In the 'Domain name' field type your `*.domain.com` where you should replace 'domain.com' by your own domain name
6. Click 'Add another to this certificate' and type `domain.com`
7. Click 'Next' and after that click on 'Review'
8. Finally click on 'Confirm and request'
9. Now, click on 'Continue' as your certificate request has been sent

Now the certificate needs to validated. That is done in the following manner:

1. Under the warning 'Validation not complete', click on `*.domain.com`
2. In this newly opened pane, click on 'Create record in Route 53'
3. Hit 'Create'

All you have to do now is wait for the certificate to be confirmed. This process can take up to 30 minutes, but for me it took around 2 minutes. Grab a coffee and chances are high that as you come back and reload the page that your certificate will be issued.

## Part 5: Configure SSL for CloudFront

As you have just configured both your CloudFront setup as well as your SSL certificate creation, the two can now be 'connected':

1. Go to [CloudFront](https://console.aws.amazon.com/cloudfront/home)
2. Click on the ID of your distributions (the fact that it is possibly not yet deployed does not matter)
3. Click on 'Edit'
4. Next to **SSL Certificate**, hit the radio button saying 'Custom SSL Certificate (example.com)' and select your freshly configured SSL certificate
5. Scroll down and click on 'Yes, Edit'

## Part 6: Connecting CloudFront to your domain for your visitors

Now, as you are probably still waiting on the deploying of your CloudFront Distribution, we will setup CloudFront so that users visiting your website will be served with content straight from CloudFront:

1. Go back to [Route 53](https://console.aws.amazon.com/route53/home)
2. Click on 'Hosted Zones' and click on the correct domain name
3. Hit the top blue button, saying 'Create Record Set'
4. In the pane on the right, select 'Yes' next to **Alias**
5. Click in the 'Alias Target' file and in the options that present themselves, scroll down to your CloudFront Distributions and select it
6. Hit 'Create'
7. Do steps 3-6 again, but now make sure to change the value in the **Name** field to 'www' and the **Alias Target** to `domain.com`

## Part 7: Redirect to index.html

Now that everything has been setup, you can test your website by going to `domain.com/yourfile.extension`. Verify that the content is served over SSL by looking at the lock icon in your browser.

However, there now is one issue: if we go to `domain.com` we are not automatically presented with tthe root `index.html` (if there is one). The same behaviour is present for subfolders.

In this final part, we are going to solve this issue:

1. Go to the [AWS Lambda](https://console.aws.amazon.com/lambda/home) control panel
2. Make sure that you select 'US East (N. Virginia)' as your location
3. Click on the orange 'Create Function' button
4. In the 'Function name' field enter `serveIndex`
5. Click 'Create function'
6. On this new page, scroll down to the code editor and paste the following code:

```javascript
'use strict';
exports.handler = (event, context, callback) => {
    
    // Extract the request from the CloudFront event that is sent to Lambda@Edge 
    var request = event.Records[0].cf.request;

    // Extract the URI from the request
    var olduri = request.uri;

    // Match any '/' that occurs at the end of a URI. Replace it with a default index
    var newuri = olduri.replace(/\/$/, '\/index.html');
    
    // Log the URI as received by CloudFront and the new URI to be used to fetch from origin
    console.log("Old URI: " + olduri);
    console.log("New URI: " + newuri);
    
    // Replace the received URI with the URI that includes the index page
    request.uri = newuri;
    
    // Return to CloudFront
    return callback(null, request);

};
```

7. Click on the orange 'Save' button in the right top corner
8. Scroll down and look for 'Execution role'
9. Click on the drop down which says 'Use an existing role' and select 'Create a new role from AWS Policy templates'
10. Enter a name for your role, for example `indexServer`
11. Again, hit 'Save'

Now, before the next couple of steps can be done, we have to allow our newly created user to deploy to Lambda Edge, so we will fix that now:

1. Go to  [IAM](https://console.aws.amazon.com/iam/home?region=us-east-1#/roles)
2. On the left side, click on 'Roles'
3. Find the newly created role and click on it
4. In this new window, click on 'Trust relationships'
5. Now, hit the blue 'Edit trust relationship' button
6. Replace the code there with the following code:

```json
{
   "Version": "2012-10-17",
   "Statement": [
      {
         "Effect": "Allow",
         "Principal": {
            "Service": [
               "lambda.amazonaws.com",
               "edgelambda.amazonaws.com"
            ]
         },
         "Action": "sts:AssumeRole"
      }
   ]
}
```

7. Now save the trust policy by clicking on 'Update Trust Policy'

The user role issue has now been fixed, so now we will continue finishing our Lambda redirect function:

1. Go back to your [Lambda function](https://console.aws.amazon.com/lambda/home?region=us-east-1#/functions)
2. Look for the 'Add triggers' list on the left side of the 'Designer'
3. Scroll down the list and select 'CloudFront'
4. Scroll down and click 'Deploy to Lamba@Edge'
5. Choose your newly created distribution and check the **Confirm deploy to Lambda@Edge** box

Now, you will have to wait for you CloudFront distribution to redeploy. As soon as that is finished,  users, when they go to `domain.com` or `www.domain.com` or `domain.com/`, will now all see the same page, without the URL being changed to `domain.com/index.html`.

## Next steps

Since you whole web hosting is completely set up now, you can stop right here with the manual. However, if you want to integrate Google Analytics for usage statistics for your site, or if you want use custom email on your account (someone@example.com) you can also follow the steps in the next two parts to configure that.

Part 10 will contain information on how to set up your own filehost/cloud in S3, which is very interesting if you are planning on storing a bunch of files in AWS.

## Part 8: Configuring Google Analytics

If you are like us and want to see who is visiting your website, which pages are being used most frequently and where your possible clients are coming from, you can set up Google Analytics to track all of this information. In the next couple of steps, we are going to set up Analytics and configure it for your website:

1. Go to the [Google Analytics website](https://analytics.google.com/analytics/web/)
2. Sign in with your Google Account
3. Click on 'Sign Up'
4. Fill in your account name and your website name
5. Under 'Website URL', select 'https://' instead of 'http://'
6. In the field next to 'https://', fill in your domain name (i.e. `domain.com`)
7. Select your industry and choose your time zone
8. Uncheck all of the boxes if you want your users to have some form of privacy
9. Now hit the blue button saying 'Retrieve Tracking ID'
10. Accept the Terms of Service

Now, your Analytics has been set up. The next step is to connect it to your website.

1. Copy the code under **General site tag (gtag.js)**
2. Go to your local `index.html` and in paste the code before the </head> tag
3. Upload the new `index.html` file to the root of your S3 bucket

Now, Google Analytics has been set up. However, it will take some time for your CloudFront distribution to start serving the newest files, so we will take care of that right now:

1. Go to [CloudFront](https://console.aws.amazon.com/cloudfront/home?region=us-east-1)
2. Click on your distribution ID
3. Click on the 'Invalidations' button
4. Hit the blue 'Create invalidation' button
5. In the field, type `*`
6. Click 'Invalidate'

Wait for the invalidation to complete and voila, your Analytics is set up. If you now go to Google Analytics, you can track the usage of your website in real time.

If you don't see your own browser visiting Analytics showing up in the browser, consider cleaning up your browser cache.

## Part 9: Setting up email

This part will show how to setup simple email on AWS using Amazon SES. It is assumed you already have setup domain on AWS at this point. *All the examples will be given for  `example.com` and S3 buckets are to be located in Ireland (`eu-west-1`). However this is just example, and to adjusted by you accordingly.* 

 1. Switch [SES](https://eu-west-1.console.aws.amazon.com/ses/home?region=eu-west-1#), and <u>switch to region where your buckets are stored</u>. (eg. Ireland). 

       - Navigate to  'Domain' on the left. Press verify your domain, use domain you have already setup at Route 53, and it will offer to add new records automatically, accept it. Verification won't take more than 2 minutes.
       - At some point it will be prompted to automatically create MX records in Route 53, agree to that.
       - Navigate to 'Email Addresses'.  Press 'Verify a new email address' use main email you are wishing to connect. (Main email where emails  received by your domain are to be forwarded). Click activation link in your main email. Return to SES, status should be "verified" now.
       - Navigate to "Rule sets", press "View active rule set" or "Create rule set" (whatever available), then press "Create Rule". 
       - First add emails account that You want to exist. (Leaving this blank will allow any emails within domain to "exist"!). Set action S3, and create a new bucket for emails (eg. '`example-com-mails`'). SES will automatically set required permissions. Name the rule, save, exit. 

 2. Switch to [Route 53](https://console.aws.amazon.com/route53/home), make sure there is  'MX' record for your domain. If there isn't press "Add record' , type 'MX'. Set value to `"10 inbound-smtp.eu-west-1.amazonaws.com"`. *If your region is not Ireland, refer to AWS help for other region's addresses.*

 3. At this point emails should be receivable. Try to send an email the address(s) you have specified in Receipt rule. Email should be stored in newly created bucket, under unique and very weird name (`sf542d3154sd31ys3d.....`).  You can download this file, and open in notepad.

 4. Switch to [AWS Lambda](https://console.aws.amazon.com/lambda/home) ,an automatic email forwarding rule should be added

       - Switch region to 'Ireland'.

       - Create a new Lamda function (eg 'mail-forward'), use 'blank function' , skip triggers setup. 

       - Navigate to <https://github.com/arithmetric/aws-lambda-ses-forwarder/blob/master/index.js> copy code.

       - Paste code in"Edit code inline" window in Lamda console. Set 'Runtime' to `'Node.js 8.10`'.

       - Navigate to line 30 and configure individual settings. 

            1. `'fromEmail'` is email from which you will receive forwarded messages - say forward@example.com
            2.  `'subjectPrefix'` and `'emailKeyPrefix'` to be left blank, thus "".
            3. `'emailBucket`' is your newly created email bucket. 
            4. `'forwardMapping'` specifies forward options. If you want all of the emails of your domain to be forwarded to one email change it to " `'@example.com' : forwarToThisEmail@yourMainMailDomain.com`"

       - Scroll down to "Execution role", leave it 'Use existing role', press "view the *nameOfRule* role on the IAM console". We need to give Lamda permissions to read emails from bucket. Press the hyperlinked policy name - "edit policy" - "JSON". Replace with the following: 


          ```json
            {
                "Version": "2016-03-04",
                "Statement": [
                   {
                      "Effect": "Allow",
                      "Action": [
                         "logs:CreateLogGroup",
                         "logs:CreateLogStream",
                         "logs:PutLogEvents"
                      ],
                      "Resource": "arn:aws:logs:*:*:*"
                   },
                   {
                      "Effect": "Allow",
                      "Action": "ses:SendRawEmail",
                      "Resource": "*"
                   },
                   {
                      "Effect": "Allow",
                      "Action": [
                         "s3:GetObject",
                         "s3:PutObject"
                      ],
                      "Resource": "arn:aws:s3:::YOUR-S3-BUCKET-NAME/*"
                   }
                ]
            };  
          ```

       - Do not forget to replace your bucket name! Save everything close IAM, return to Lamda.

       - In "Basic Settings" set timeout to 5 seconds, leave memory 128mb. Save everything, close Lamda console.

 5. Switch to [SES](https://eu-west-1.console.aws.amazon.com/ses/home?region=eu-west-1#). Open the Receipt Rule You have created in step #1d

       - Right after S3 action, add another one, select type 'Lamda' 
       - Chose newly created Lamda function. If it won't appear, most probably you messed up regions. Remember, Lamda function, email bucket and SES must be configured to the same location!

 6. At this step it would be good to send a test email again, it should now be forwarded to specified email address.

 7. Navigate to 'SMTP Settings'. Press  "Create my SMTP Credentials" and store this file safe. 

       - Don't leave this page just yet, because you will need "server name" from top of it to connect it to main email. 
       - Open Gmail, navigate Settings==>Accounts and Import==>Send email as: ==> Add another email account
       - In the following pop up specify email address you have created in #3. Press 'next'. Specify SMTP server as stated on SES SMTP page, and use downloaded SMTP credentials to login
       - You should now receive forwarded email with confirmation code. Input it and you are done!

 8. Final step - exist 'sandbox'. AWS limits email capabilities and won't  let you send any emails to anyone expect those that were verified. Mainly to avoid spam.

       -  Navigate to "Sending statistics", and press "Request a Sending limit increase". 
       - Input your domain name, tick all 3 ticks (although they say they are optional, they aren't). 
       - Select limiting factor (eg #of emails per 24h, set to 200). 
       - Shortly describe why you need it. This is very general, mainly to prove you are not a robot and this is a real website with real owner. AWS takes about 1 business day to respond. 

Wait for AWS to remove 'sandbox' restriction. Once it done, your email system setup is finished.

## Part 10: Setting up file browser

At some point you might want to use S3 bucket (same, or another one) as web-accessible public file storage. You can either access specific file by end link eg. `www.domain.com/folder/file.type` or create a file browser, an `index.html` which will display all files and directories automatically in user friendly form eg. navigating to `www.domain.com/folder` will show all files in there. 

     	1. Download <https://github.com/nolanlawson/pretty-s3-index-html/blob/master/index.html> and put it in desired directory.
     	2. Line 92 => change  `var bucketUrl = document.location.href.replace(/\/[^\/]+$/, '');`' to `var bucketUrl = "https://s3-REGION.amazonaws.com/BUKCET';`, eg. `var bucketUrl = "https://s3-eu-west-1.amazonaws.com/example.com';`
     	3. Line 122 => change  `self.url = bucketUrl + '/' + self.key`;  to  `self.url = b '/' + self.key;`
     	4. Line 128 => change "`Unknown`" to "". This will remove showing "Unknown" as type for folders.  (Don't know why to be honest, some bug in script)
     	5. Line 136 => change  '`var Title`' to desired title
     	6. Make sure bucket has public access rights  (described before) and there is a EdgeLamda function to open `index.html` automatically (Part 7)

At this point when navigating to your folder/bucket eg. `example.com/filhost` u should see directory listed. In case you would like to setup file hosting in separate to be accessible within domain, for example [`filehost.example.com`](), you need the following:

 	1. Create separate S3 bucket, make it public, add public READ and LIST permissions
 	2. In the properties of bucket, select "Host a website", and add index.html (doesn't exist yet, it's fine) as root object.
 	3. Create a new CloudFront distribution for this bucket, connected to new subdomain (eg [`filehost.example.com`]()), and add index.html as default object. If you would like to set up SSL, don't forget to claim new HTTPS certificate!
 	4. After deployment of CloudFront distribution, switch to Route 53, and add 2 new records ()[filehost.example.com]() and www.filehost.example.com) pointing at new CloudFront deployment ID
 	5. Add a new EdgeLamda function to invoke index.html, triggered by <u>new</u> CloudFront deployment
 	6. Follow steps 1-4 from beginning of this chapter to set up and add index.html to your newly created bucket.

## Conclusion and final thoughts

Finally: you have your own website hosted at AWS! As you have seen, it is not super hard or complicated to setup your own web hosting at AWS, it is just a long list of steps to go through.

One you can do after you have followed this guide is installing the [AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/install-bundle.html) (command line interface), so you can upload a bunch of files, or for example, the newest version of your website, over the command line with a single command, for example:

```bash
aws s3 sync savetimetable s3://domain.com/folder/ --exclude ".git/*" --exclude ".gitignore" --exclude "README.md" --delete
```

This simple command line syncs your complete working folder with a folder on S3 without including all of git information. The `--delete` takes care of deleting older versions of the files on S3.

We hope you found it easy to follow this guide, as that was the sole intention of this writing this piece! If you have any questions left, you can contact us via [hi@douwe.xyz](mailto:hi@douwe.xyz) and [alex@vinalex.net](mailto:alex@vinalex.net).