---
title: "Basic DNS and Hosting Configuration"
date: 2021-01-16T10:50:00-04:00
draft: false
toc: false
images: ["img/posts/dns/og-image.png"]
tags: 
  - programming
  - dns
  - aws
  - netlify
  - github
keywords: [programming, dns, aws, netlify, github]
description: "This post provides a few example configurations for different DNS and hosting providers."
---

# Introduction

This post is not much more than some notes I recently took when getting a couple simple static sites set up with custom domains in the form of a few examples.
This use case is very common, and with some configurations it is very easy.
Other configurations were a bit more complicated than I'd previously remembered them being, and configuration is a bit different across across the various DNS and hosting providers.
**Each of these examples shows how to:**

1. Use a custom second level domain with a website
2. Facilitate using a `www` subdomain
3. Use SSL

In this post I share a few configurations using common domain registrars, DNS and hosting providers.
The first step is going to the domain registrar of choice, such as [Google Domains](https://domains.google/), [AWS Route 53](https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/domain-register.html), or one of the many others.
Generally these are my two registrats of choice, primarily because I use other services they provide, like [Google Workspace](https://workspace.google.com/) or many of the services offered by AWS.
The examples below are with Google Domains.

After registering the custom domain, the site needs to be hosted.
The following examples use Netlify and GitHub pages.
Note that the examples below are not the only combinations of configurations using these different services.
They just show a sample of what I've done for a few projects and can be adapted as needed.

# Hosting with Netlify DNS

I've really enjoyed using [Netlify](https://www.netlify.com/) for simple projects.
Netlify makes it really simple to connect to a GitHub repository and configure deploys.
The easiest way to use a custom domain with Netlify is using Netlify's DNS.
You just need to specify the name servers with your domain registrar.

<img src="/img/posts/dns/netlify-name-servers.png" width="600" style="border-style:solid;border-width:1px;border-color:rgb(218,220,224);border-radius:6px;box-shadow:rgba(14,30,37,0.12) 0px 2px 4px 0px;"/>

In Google Domains DNS settings this is very easy.

<img src="/img/posts/dns/google-domains-netlify-nameservers.png" width="600" style="border-style:solid;border-width:1px;border-color:rgb(218,220,224);border-radius:6px;"/>

Now in Netlify you can add DNS records.

<img src="/img/posts/dns/netlify-dns-records.png" width="600" style="border-style:solid;border-width:1px;border-color:rgb(218,220,224);border-radius:6px;box-shadow:rgba(14,30,37,0.12) 0px 2px 4px 0px;"/>

And configure your sites custom domains.

<img src="/img/posts/dns/netlify-custom-domains.png" width="800" style="border-style:solid;border-width:1px;border-color:rgb(218,220,224);border-radius:6px;box-shadow:rgba(14,30,37,0.12) 0px 2px 4px 0px;"/>

Note in the above configuration the `www` subdomain is configured to redirect to the primary second level domain, which is a common desirable configuration.
Finally, SSL configuration is seamless through Netlify with LetsEncrypt, allowing a certificate to be configured for both the second level domain and the subdomain.

<img src="/img/posts/dns/netlify-ssl.png" width="700" style="border-style:solid;border-width:1px;border-color:rgb(218,220,224);border-radius:6px;box-shadow:rgba(14,30,37,0.12) 0px 2px 4px 0px;"/>

This is about as straightforward of a configuration one can get, and a default choice of mine when spinning up a simple static site.
However, if you use Google Workspace or other Google Products, it may be more convenient to use Google's name servers instead.
See for example [Set up Google Workspace with a third-party DNS host](https://support.google.com/a/answer/6398669).
The next section shows how to do this while keeping the site hosted on Netlify.

# Hosting on Netlify with Google DNS

The first step is simple: change the check the box to use Google Domains name servers instead of Netlify's.

<img src="/img/posts/dns/google-domains-google-nameservers.png" width="600" style="border-style:solid;border-width:1px;border-color:rgb(218,220,224);border-radius:6px;"/>

The custom domain (including the `www` subdomain) configuration when using another DNS provider is identical to the case above when using Netlify's DNS.

<img src="/img/posts/dns/netlify-custom-domains-external-dns.png" width="600" style="border-style:solid;border-width:1px;border-color:rgb(218,220,224);border-radius:6px;box-shadow:rgba(14,30,37,0.12) 0px 2px 4px 0px;"/>

The SSL configuration is identical as well.

<img src="/img/posts/dns/netlify-ssl.png" width="700" style="border-style:solid;border-width:1px;border-color:rgb(218,220,224);border-radius:6px;box-shadow:rgba(14,30,37,0.12) 0px 2px 4px 0px;"/>

The DNS records are configured through Google Domains as shown below.
The Netlify Docs [Configure external DNS for a custom domain](https://docs.netlify.com/domains-https/custom-domains/configure-external-dns/) describe the necessary configuration, which includes an `A` record which points to Netlify’s load balancer at `104.198.14.52`.

<img src="/img/posts/dns/google-dns-with-netlify.png" width="800" style="border-style:solid;border-width:1px;border-color:rgb(218,220,224);border-radius:6px;"/>

The `CNAME` record is used to point to the subdomain, which Netlify redirects automatically to the primary domain, as specified in the Netlify domain configuration settings.
I think the most helpful point to note here that I didn't immediately realize when rushing through my first configuration, is that unlike Route 53, the second level record names in the Google Domains resource records display as `@` instead of the domain itself.
This is described in the [About resource records](https://support.google.com/domains/answer/3251147) help page.

# Hosting on GitHub Pages with AWS Route 53 DNS

The first step again is to use configure the settings for your domain in Google Domains to use the AWS nameservers.
These are provided by AWS when creating a new hosted zone and should be entered into the Google Domains DNS configuration as shown below.

<img src="/img/posts/dns/google-domains-aws-nameservers.png" width="600" style="border-style:solid;border-width:1px;border-color:rgb(218,220,224);border-radius:6px;"/>

Next, the GitHub Pages configuration needs to be set, as that is where the site will be hosted.
The basic configuration page is quite straightforward.

<img src="/img/posts/dns/github-pages-config.png" width="700"/>

The GitHub docs pages [About custom domains and GitHub Pages](https://docs.github.com/en/github/working-with-github-pages/about-custom-domains-and-github-pages) and [Managing a custom domain for your GitHub Pages site](https://docs.github.com/en/github/working-with-github-pages/managing-a-custom-domain-for-your-github-pages-site) provide all the needed information to configure all the settings.
Specifically we first follow the instructions under 
[Configuring an apex domain](https://docs.github.com/en/github/working-with-github-pages/managing-a-custom-domain-for-your-github-pages-site#configuring-an-apex-domain) so that a user who navigates to [https://mysamplesite.com](https://mysamplesite.com) is brought to our site.
Note that adding SSL is done via a simple checkbox in the GitHub Pages settings.
The IP addresses are given in the GitHub docs above, and it's straightforward to add the necessary `A` record to Route 53.

<img src="/img/posts/dns/aws-route-53-config.png" width="800" style="border-style:solid;border-width:1px;border-color:rgb(218,220,224);border-radius:0px;"/>

Now we need to configure the `www` subdomain to also send users to our site, and have an SSL certificate.
This is where the solution doesn't seem as straightforward as it should be.
In trying to figure out a solution, I came across the issue [GitHub Pages: Generate SSL certificate for www subdomain when a custom domain is set to an apex (and vice versa)](https://github.com/isaacs/github/issues/1675) that described exactly this problem.
There was some good discussion and a list of popular websites that are broken because of the issue of getting an SSL certificate for the `www` subdomain, such as the one below for [https://www.scikit-learn.org](https://www.scikit-learn.org).

<img src="/img/posts/dns/scikit-learn-ssl.png" width="600" style="border-style:solid;border-width:1px;border-color:rgb(238,240,244);border-radius:6px;box-shadow:rgba(14,30,37,0.12) 0px 2px 4px 0px;"/>

It turns out this solution was quite easy in Google Domains with a synthetic record.

<img src="/img/posts/dns/google-domain-synthetic-record.png" width="800" style="border-style:solid;border-width:1px;border-color:rgb(218,220,224);border-radius:6px;"/>

However, in this particular project I wanted to use AWS Route 53 to manage DNS, and so this wasn't the solution I was looking for.
I figured there would be an equivalently easy solution with Route 53.
It turns out that wasn't quite the case, but the steps are fairly straightforward and described below.

1. Create a certificate for the subdomain using AWS [Certificate Manager](https://aws.amazon.com/certificate-manager/)<br><br>
<img src="/img/posts/dns/aws-acm.png" width="800" style="border-style:solid;border-width:1px;border-color:rgb(218,220,224);border-radius:0px;"/><br>
2. Create an S3 bucket and enable static website hosting.<br><br>
<img src="/img/posts/dns/aws-s3-static-website-hosting.png" width="500" style="border-style:solid;border-width:1px;border-color:rgb(218,220,224);border-radius:0px;"/><br>
3. Create a CloudFront distribution.
Associate the certificate with this CloudFront distribution.<br><br>
<img src="/img/posts/dns/aws-cloudfront-general.png" width="800" style="border-style:solid;border-width:1px;border-color:rgb(218,220,224);border-radius:0px;"/><br>
Set the S3 bucket as the origin of the distribution.<br><br>
<img src="/img/posts/dns/aws-cloudfront-origin.png" width="800" style="border-style:solid;border-width:1px;border-color:rgb(218,220,224);border-radius:0px;"/><br>
4. Create an `A` record in Route 53 for the `www` subdomain, in this case `www.mysamplesite.com` and route traffic to the CloudFront distribution.<br><br>
<img src="/img/posts/dns/aws-route-53-config-www.png" width="800" style="border-style:solid;border-width:1px;border-color:rgb(218,220,224);border-radius:0px;"/>

This completes the configuration, now redirecting traffic from the `www` subdomain to the second level domain, with SSL.
Given how common this use case is though, it seems like AWS would make this as easy as Google did with their synthetic records.
Perhaps there is a better way that I'm just not aware of.
