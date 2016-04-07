+++
date = "2016-04-07T11:32:53+02:00"
draft = false
title = "A Hugo blog - automated"

+++

# A Hugo blog - automated - Howto.

## The big Why?

I have to admit I am annoyed by CMS systems. All of them require constant care and are usually hard to use. In the past I have tried [Wordpress](http://www.wordpress.com) or [Drupal](http://www.drupal.com) and always found that they kind of work, but once I left it alone for a couple months it got ugly. So I got rid of all these installations after a while.

Currently I only have to care for one CMS called [Contao](http://www.contao.org) which gives me constant grief by being hard to use and requiring the usual attention.

So I stumbled across the Static Site Generators a while ago. It started by reading about [Flask](http://flask.pocoo.org/) and the [Lektor](https://www.getlektor.com/) tool by the same author. While I am still a big fan of all things [Python](https://www.python.org/) I wanted to try [Go](https://www.golang.org) and therefore looked at the [List of Static Site Generators](https://www.staticgen.com/) and found [Hugo](https://gohugo.io/).

Now the next step was to actually have a simple way of running a Hugo site. I found an interesting Github repo by Nathan Youngman [Link](https://github.com/nathany/hugo-deploy) which offered an easy way of deploying automatically from Github into S3 using CircleCI. The only thing that was really missing: **A guide that walks through all the steps.**

So here it is. My steps and all the gotchas that I found.

## Overview

1. Install Hugo locally for testing and playing with it - free
2. GitHub account (which I already had) - free
3. CircleCI account - which connects to GitHub- free
4. AWS account with a configured S3 bucket (cheap)
5. Cloudflare DNS for HTTPS - because we would like to join Encryption Everywhere right? - free

### Installing Hugo

Piece of cake. Follow the [Instructions](https://gohugo.io/overview/installing/) on the Hugo site and be done with it. Homebrew on Mac works beautifully.

### GitHub account

Come one you probably have at least one you can play with. Otherwise you would not read this blog. :)

### CircleCI

The [CircleCI](https://circleci.com/) site makes it extremely simple to add your GitHub account and use it as the reference for creating projects. One project in one container is free for hobbyists like us.

> **Note**: If you are using the [Repo](https://github.com/nathany/hugo-deploy) including the circleCI config files there are two **unmentioned** Environment variables that need to be set.
> 1. BUCKET - the bucket name that should be used in S3
> 2. REGION if you want to deploy anywhere else than us-east-1. In my case I wanted to deploy to Frankfurt, but that resulted in an error. See the S3 section for more information.

> These can be set in your project details under *build settings*.

### AWS S3 bucket

I got a free Amazon AWS account a while ago to play with certain [aspirations](http://lg.io/2015/07/05/revised-and-much-faster-run-your-own-highend-cloud-gaming-service-on-ec2.html) for Gaming on AWS. I never followed through with that, but I have this account already setup. So the next step was to create a S3 bucket for the static site. Couple of things to keep in mind:
1. Name the bucket exactly as the hostname you want to use it from. In my case it is *www.treutler.cc*.
2. Choose your region wisely. I **encountered a problem** not being able to connect to *eu-central-1* S3 bucket with the supplied tools, because it only supports the authentication v4, which the toolchain that Nathan uses does not support. So I had to switch to *eu-west-1* instead. If the toolchain gets updated to use the new SDK by Amazon it should work with the *eu-central-1* as well.
3. Enable the static site hosting in the Properties of the bucket.
4. Enable CORS in the same page.
5. Generate a new [IAM](https://aws.amazon.com/iam/) user in the AWS portal and add the Access Key and the Secret Access key to CircleCI (AWS permissions).
> **Note:** Don't forget to actually give Full S3 permissions to that user! Otherwise you will get only "Access denied" when it tries to deploy your site.

### Cloudflare

Cloudflare can be used setup DNS for your site. Follow the Cloudflare [Instructions](https://www.google.de/url?sa=t&rct=j&q=&esrc=s&source=web&cd=1&cad=rja&uact=8&ved=0ahUKEwiQ3o7dtvzLAhVGwBQKHTYTAsgQFggdMAA&url=https%3A%2F%2Fsupport.cloudflare.com%2Fhc%2Fen-us%2Farticles%2F200168926-How-do-I-use-CloudFlare-with-Amazon-s-S3-Service-&usg=AFQjCNFmJUE0Kv2eW0TQONhg2P9gOXZhHQ&sig2=dHbmZHY401-_aN4LGQ7XbA) to setup the service with your S3 account. Here the naming of the bucket will be important to match your domain.
> **Note:** Keep in mind that you have built a cache in front of your website. So your *update* in the repo might not be there instantaneously.

### Example hugo-deploy repo

So I took the Repo by Nathan [Link](https://github.com/nathany/hugo-deploy) and put the code I wanted into a new [Repo](https://github.com/yogitea/hugo-test).
Next step was to create the circleCI project and commit changes to the repo to trigger the built process.
Once I had found all the **issues** that prevented the built it actually work. Now the hard part begins... Creating content ;)

> Issues I had:
> 1. Missing (non documented) Environment variables for built in CircleCI.
> 2. S3 eu-central-1 not supported with tool chain. Had to switch to eu-west-1. Authentication error.
> 3. Missing IAM permissions to access S3. (Simple oversight)
> 4. Default deployment tool s3up command line did not include a region, which I needed to add for any custom region (default us-east-1) to work. The error is that bucket is not available.

    Example change circle.yml
        - s3up -source=public/ -bucket=$BUCKET -region=$REGION


## Conclusion

It worked. You are reading the this blog entry and it was written in Atom using Markdown, pushed to GitHub, automatically built by CircleCI, automatically uploaded to S3, published by Cloudflare DNS.

It could not get easier than this, and it does not have any security vulnerabilities built into it like all the CMS out there.

> Simply *AWESOME*
