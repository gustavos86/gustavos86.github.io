---
title: Creating a domain in AWS Route 53 and point it to GitHub Pages
date: 2023-09-16 09:47:00 -0700
categories: [AWS, Route 53]
tags: [aws, route 53]     # TAG names should always be lowercase
---

![]({{ site.baseurl }}/images/services/jekyll-logo-dark-solid.png)

![]({{ site.baseurl }}/images/services/route53.png)

This is my very first post in this blog I have created to document my progress toward learning about public cloud providers.
I am a firm believer in "learning by doing" to help me learn more quickly and it is for this reason that I am starting today with this blog.
Today, I am registering a new domain in AWS Route 53, creating the necessary records, and pointing them to this blog. The blog has been created using [Jekyll](https://jekyllrb.com/) as a static site generator (written in Ruby) and it is hosted in [GitHub Pages](https://pages.github.com/) for free.

Creating this blog in GitHub Pages following and the documentation (see [Types of GitHub Pages sites](https://docs.github.com/en/pages/getting-started-with-github-pages/about-github-pages#types-of-github-pages-sites)) required the original URL to be exactly `<username>.github.io`.
I wanted to have my own domain and I am using AWS Route 53 to register **myjourneytocloud.com**. I am documenting the steps that were taken to accomplish this using the AWS Management Console.

1. Starting from the Route 53 service in the AWS Management Console initiate the wizard

Select **Register a domain**

![]({{ site.baseurl }}/images/2023/09-16-Creating-a-Domain-in-Route-53/01-Route-53-initiate-the-wizard.png)

2. Look to see if the domain is available. In my case, **myjourneytocloud.com** was already taken.
However, I noticed **myjourneytocloud.net** is available so took it.
Select the desired domain and click on the button that shows up **Proceed to checkout**

![]({{ site.baseurl }}/images/2023/09-16-Creating-a-Domain-in-Route-53/02-Route-53-checking-availability.png)

![]({{ site.baseurl }}/images/2023/09-16-Creating-a-Domain-in-Route-53/03-Route-53-available-domains.png)

3. The next screen asks for how long you want to register the domain and it reflects the cost of it. This screen also shows the option to enable or disable auto-renew. In my case, I chose  1 year and unchecked the **Auto-renew** checkbox.

![]({{ site.baseurl }}/images/2023/09-16-Creating-a-Domain-in-Route-53/04-Route-53-duration-auto-renew.png)

4. The next screen asks for Contact Information such as name, email, phone number, address, Country, State, City, Zip code / Postal code and a few privacy options such as:

    - Admin Contact (enable|disable) - *Same as registrant contact*
    - Tech contact (enable|disable) - *Same as registrant contact*
    - Privacy contact (box to check|uncheck) - *Turn on privacy protection for all contacts of myjourneytocloud.net. When you turn on privacy protection, contact information is hidden from WHOIS queries.*

![]({{ site.baseurl }}/images/2023/09-16-Creating-a-Domain-in-Route-53/05-Route-53-contact-information.png)

5. Finally, the **Review and submit** section shows all the information entered to be confirmed. Just click on the appropriate button to complete the process of registering a new domain if all looks good.

![]({{ site.baseurl }}/images/2023/09-16-Creating-a-Domain-in-Route-53/06-Route-53-review-and-submit.png)

> As a note, [Route 53 is primarily a global service](https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/disaster-recovery-resiliency.html)

6. To see the status of creating the new Domain, on the left navigation bar go to **Domains > Requests**. The domain registration should be in progress.

![]({{ site.baseurl }}/images/2023/09-16-Creating-a-Domain-in-Route-53/07-Route-53-requests-in-progress.png)

In my case, I had to update the credit card information in the root account. Once done I repeated the process of registering the domain.

![]({{ site.baseurl }}/images/2023/09-16-Creating-a-Domain-in-Route-53/08-Route-53-requests-in-progress_2.png)

During this process, I received a few emails in my inbox:

    - One from my bank letting me know that a charge was done to my credit card
    - Another email with the invoice from AWS
    - Another one from AWS including a link to verify the email address I put when registering the domain (step 4)
    - A fourth email from AWS informing me that my email was successfully verified for domain registration
    - An finally, an email from AWS letting me know the registration of the domain succeeded

7. Once the Domain has been fully created, on the left navigation bar go to the **Hosted Zone** section and open the existing Hosted Zone to create two new **Records**. The existing Hosted Zone was created when the domain process was completed.

![]({{ site.baseurl }}/images/2023/09-16-Creating-a-Domain-in-Route-53/10-Route-53-aws-creating-hosted-zone.png)

Click on **Create record**

![]({{ site.baseurl }}/images/2023/09-16-Creating-a-Domain-in-Route-53/12-Route-53-aws-creating-records.png)

8. To create the record, you the DNS IP addresses of GitHub which are found in GitHub's documentation section [Configuring an apex domain](https://docs.github.com/en/pages/configuring-a-custom-domain-for-your-github-pages-site/managing-a-custom-domain-for-your-github-pages-site#configuring-an-apex-domain)

![]({{ site.baseurl }}/images/2023/09-16-Creating-a-Domain-in-Route-53/13-Route-53-github-getting-ip-addresses-for-a-records.png)

9. Create the first record as follows:

    - Leave the subdomain field *blank*
    - Record type is A
    - Value is all the IP addresses of GitHub DNS

![]({{ site.baseurl }}/images/2023/09-16-Creating-a-Domain-in-Route-53/14-Route-53-aws-creating-first-record.png)

10. Create the second record as follows:

    - In subdomain type *www*
    - Record type is CNAME
    - Value is `<username>.github.io` for your GitHub account (in my case `gustavos86.github.io`)

![]({{ site.baseurl }}/images/2023/09-16-Creating-a-Domain-in-Route-53/15-Route-53-aws-creating-second-record.png)

11. The result should look similar to the following screenshot

![]({{ site.baseurl }}/images/2023/09-16-Creating-a-Domain-in-Route-53/16-Route-53-aws-records-created.png)

12. In your GitHub repo (in my case [https://github.com/gustavos86/gustavos86.github.io/](https://github.com/gustavos86/gustavos86.github.io/)) go to **Settings > Pages**

Under section **Custom Domain** add your DNS and click on the **save** button.

![]({{ site.baseurl }}/images/2023/09-16-Creating-a-Domain-in-Route-53/09-Route-53-github-custom-domain.png)

Wait for the validation to complete. It should take just a few seconds.

![]({{ site.baseurl }}/images/2023/09-16-Creating-a-Domain-in-Route-53/17-Route-53-github-checking-domain.png)

The result should look like the following screenshot. Make sure you check the **Enforce HTTPS** checkbox as well.

![]({{ site.baseurl }}/images/2023/09-16-Creating-a-Domain-in-Route-53/18-Route-53-github-custom-domain-ok.png)

This should have created a file named **CNAME** in the project main folder. In my case [https://github.com/gustavos86/gustavos86.github.io/blob/main/CNAME](https://github.com/gustavos86/gustavos86.github.io/blob/main/CNAME) and the content is just:

```
myjourneytocloud.net
```

## Common error

13. If you are getting a blank page with raw text of index.html file or similar when visiting your site

![]({{ site.baseurl }}/images/2023/09-16-Creating-a-Domain-in-Route-53/20-Common-error.png)

What you have to do is change the **GitHub Pages** setting to **Build and deployment** **Source** : **GitHub Actions**

![]({{ site.baseurl }}/images/2023/09-16-Creating-a-Domain-in-Route-53/19-Route-53-github-build-and-deployment-github-actions.png)

14. Finally, if you are using [Jekyll](https://jekyllrb.com/) just like me, you may want to update the `_config.yml` file:

```
. . .
# fill in the protocol & hostname for your site, e.g., 'https://username.github.io'
url: https://www.myjourneytocloud.net
. . .
```

That's it, now https://www.myjourneytocloud.net/ and https://myjourneytocloud.net/ point to this blog.

## References

- [How to Create a Simple, Free Blog with Jekyll and GitHub Pages](https://chrisjhart.com/Creating-a-Simple-Free-Blog/)
- [How to deploy GitHub pages with AWS Route 53 registered custom domain and force HTTPS](https://medium.com/@benwiz/how-to-deploy-github-pages-with-aws-route-53-registered-custom-domain-and-force-https-bbea801e5ea3)
- [Jekyll deployed in github shows raw text of index.html file](https://stackoverflow.com/questions/72079476/jekyll-deployed-in-github-shows-raw-text-of-index-html-file)