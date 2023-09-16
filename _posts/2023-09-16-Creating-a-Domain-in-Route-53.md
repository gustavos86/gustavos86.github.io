---
title: Creting a Domain in Route 53
date: 2023-09-16 09:47:00 -0700
categories: [AWS, Route 53]
tags: [aws route 53]     # TAG names should always be lowercase
---

This is my first post in this blog which I have created to document my progress learning about the different public cloud providers starting with AWS (Amazon Web Services). I am a firm believer that "learning by doing" will help me to embrase more quickly all the benefits Public Cloud Providers have to offer.

This blog lives in my [GitHub](https://github.com/gustavos86/gustavos86.github.io) account and it is published using [GitHub Pages](https://pages.github.com/).

Creating a this blog in GitHub Pages required following GitHub Page's documentation and therefore the URL of this blog was initially created using the naming convention of `<username>.github.io` as described in [Types of GitHub Pages sites](https://docs.github.com/en/pages/getting-started-with-github-pages/about-github-pages#types-of-github-pages-sites)

I wanted something more personalized as the URL. So I am using my AWS Route 53 to create a domain that points to this blog.

I will start by directly going to the Route 53 service in my AWS account and from there initiate the wizard.

I am choosing **Register a domain**

![]({{ site.baseurl }}/images/2023/09-16-Creating-a-Domain-in-Route-53/01-Route-53-initiate-the-wizard.png)

I looked for the domain **myjourneytocloud.com** and it says it is not available.

![]({{ site.baseurl }}/images/2023/09-16-Creating-a-Domain-in-Route-53/02-Route-53-checking-availability)

I noticed **myjourneytocloud.net** is available so I will go with that one. I select it and click on the button that shows up **Proceed to checkout**

![]({{ site.baseurl }}/images/2023/09-16-Creating-a-Domain-in-Route-53/03-Route-53-available-domains.png)

Choosing 1 year and uncheck **Auto-renew** for now as that is my choice for the time being.

![]({{ site.baseurl }}/images/2023/09-16-Creating-a-Domain-in-Route-53/04-Route-53-duration-auto-renew.png)

I start filling the form which asks for my names, email, phone number, address, Country, State, City, Zip code / Postal code and a few privacy options such as:

- Admin Contact [enable|disable] - Same as registrant contact
- Tech contact [enable|disable] - Same as registrant contact
- Privacy contact [check|uncheck] - Turn on privacy protection for all contacts of myjourneytocloud.net. When you turn on privacy protection, contact information is hidden from WHOIS queries.

![]({{ site.baseurl }}/images/2023/09-16-Creating-a-Domain-in-Route-53/05-Route-53-contact-information.png)

Finally, the **Review and submit** section completing the process of registering a new domain.

![]({{ site.baseurl }}/images/2023/09-16-Creating-a-Domain-in-Route-53/06-Route-53-review-and-submit.png)

As a note, [Route 53 is primarily a global service](https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/disaster-recovery-resiliency.html)

In the navigation bar in the left side, in section **Domains > Requests** I see that registering my domain is in progress.

![]({{ site.baseurl }}/images/2023/09-16-Creating-a-Domain-in-Route-53/07-Route-53-requests-in-progress.png)

> Registration Status ended up being Failed. Here I faced an error related to payment. It seems I need to update my credit card information in my AWS root account. I logged again to this account this time using the root user to update my account information.

Once I updated my credit card information I had to repeat the process of registering the domain again.

This time, I got some emails in my inbox:

1. One email from my bank letting me know that a charge was done to my credit card
2. Another email from AWS including a link to verify the email address I put when registering the domain
3. Finally, a third email from AWS informing me that my email was successfully verified for domain registration

It is still taking in some minutes to complete the process of registering my new domain.

![]({{ site.baseurl }}/images/2023/09-16-Creating-a-Domain-in-Route-53/07-Route-53-requests-in-progress_2.png)