---
layout: page
title: "Thinking Like A Hacker — Part II: When The Backdoor Finds You"
permalink: /2017-08-14-thinking-like-a-hacker-part-2
---

The world of software engineering is the culmination of efforts — teams assembling new works, improving old works, leveraging the byproducts of other teams, internally and externally to the business. The countless number of eyes poring over most of the tools we rely on gives us a sense of comfort; code having been around for over a decade should be bulletproof, right?

![back door](https://cdn-images-1.medium.com/max/1600/0*w8wUTygI3nDBpE55.jpg "back door")
Ceci n’est pas une porte dérobée. I don’t speak French, but I do speak Google Translate.

In the less-idyllic real world, we know this isn’t true. Long-standing vulnerabilities in crucial pieces of infrastructure appear every day. At a certain point of making considerations to the components we rely on, a level of acceptable risk has to be determined. We mitigate these risks by instituting firewalls, [hoping that a product with security at its focus will never betray us.](https://communities.cisco.com/community/technology/security/ngfw-firewalls/blog/2017/04/14/equation-group-exploit-hits-newer-cisco-asa-juniper-netscreen) We also mitigate these risks by limiting our reliance on other projects, reinventing the wheel where necessary or simply not partaking the next Web 3.0 technology until it is a little more long in the tooth.

It is often the case that the most unsuspected piece of the architecture is the one that bites us in the end. A great case for this is OpenSSL. It’s frequently taken the brunt of security researchers’ ire in the past few years, but part of this blame comes from the fact that despite it being a project to provide security, few contributors to the project were taking as close an approach towards improving the overall security of the product. Nevertheless, it is used by [about two thirds of all HTTPS-serving websites.](https://www.asecurelife.com/heartbleed-bug/)

In [Part I](/2017-08-12-thinking-like-a-hacker-part-1) I spoke from a white box perspective. I have no desire to explicitly train would-be hackers, and there are far better resources to learn from if that is your aim. However, it is not always the case that you will have 100% introspection into all that inhabits your application’s ecosystem. Never mind the level of effort required even if all software used by your project were open source, but generally speaking, there will be components involved for which you simply do not have access to the code. A great example for this is the myriad of graphics, wireless and other hardware drivers used on otherwise open source environments.

So let’s revisit the first scenario from Part I. I’m going to pick on Larry’s e-commerce site again. Larry’s learned his lesson this time, and had his code thoroughly reviewed. No more vulnerabilities in his code, and he’s starting to follow best practices for managing passwords and deployments. He now even has logging on the queries that get executed, _just in case._ Now that his new and improved site is up and running, he gets to rest easy this weekend.

![nope](https://cdn-images-1.medium.com/max/1600/0*-HhmhuG0Ch9kiPH6.jpg "nope")
Not in my examples, at least.

To his dismay, he wakes up to a page in the late hours of the evening: “ALL SHOPS DOWN. CUSTOMER DATA GONE.” Larry’s going to have a very bad Sunday. Reviewing the application logs on the logging service’s portal indicates nothing has gone awry, but still, all of the data is gone. Good thing there’s backups, but if it wasn’t his code, then how? Digging a little deeper into the logs, Larry sees that an address from Romania successfully connected to the database, and then executed a DROP. “How could they have gotten the password?” Larry asks himself. Good question.

Larry RDPs into the webserver and starts up a copy of SSMS. After restoring the database and resetting all passwords, he shuts off connectivity to the database from all addresses but the web servers, a practice he will likely be reminded of on Monday as one that should have always been the case. Letting his boss know everything is back online, he heads back to bed.

![time](https://cdn-images-1.medium.com/max/1600/1*b1-g4rAKPKwkeZyToTl52Q.png "time")
Time is an illusion. Bedtime doubly so.

In our hearts, I think we all knew the story wasn’t going to end here. Larry wakes again to the chorus of panic: “ALL SHOPS DOWN. SITES ARE DEFACED.”

“You’ve got to be kidding.”

Once again, Larry opens his laptop, this time to confirm the ecommerce stores are in fact all displaying a banner of hacking domination. Once again, he goes to the logs. Some time not long after he restored the databases, the event logs on the webservers indicate a successful RDP session started from that same Romanian address, using _his credentials_. Knowing that he certainly didn’t use a password for his account for anything else, the gears start turning as he changes his password. Before anything else though, he issues an emergency redeploy of the webservice, just in case the servers themselves are compromised. Finally, he turns off RDP access to any server but his own address and the office. “Let’s see them work around that one,” he says to himself, just before trying one last time to get some shuteye.

![hacked](https://cdn-images-1.medium.com/max/1600/1*iEjlw8VHWl4MG_OWMvRuQQ.png "hacked")
They worked around that one.

Realizing that he’s not getting any more sleep this morning, Larry once again looks at the logs, and notices that this time, the site was defaced through editing store items. Swearing up and down there was no way to break the authentication and authorization logic, he looked for patterns. Not all shops had been hacked. Of those that were, all of them had been logged into by the shop account’s administrator since the website was redeployed, followed by the attacker editing their stores as the administrators minutes later. Further, the address of the attacker keeps changing; there’s no way to block them.

After a heated call with management, Larry once again restores from a backup, and sets up a temporary redirect to an outage page. “These hackers are like magicians. How can they keep doing this?” This time he’s not ruling _anything_ out. Once again, he reviews the logs, looking for anything that could be in error. After about a half hour of pulling out his hair, he notices that the third party emailing service (used to send customers email, and handle mail-based communication with shops) keeps posting service restart messages. Lining up the timestamps, it becomes apparent that the original attack started just a few hours after the email service restarts began. Having a much better idea about the source of the attack, Larry fires up Wireshark in the hopes of watching it in action. Sure enough…

![oh](https://cdn-images-1.medium.com/max/1600/1*o7SV2_RpANhV3xFiIUW0Iw.png "oh")
Oh.

The emailing service, which hasn’t seen updates in years, was using an outdated version of OpenSSL. The mail service API is compromised, sending heartbleed packets back to the service running on the webserver. This means they’ve been reading memory from the server the entire time this attack has been ongoing. It’s how they got the connection string (in memory). It’s how they got the remote desktop credentials (pass the hash attack, in memory). It’s how they got access to every shop administrator who logged in after the redeploy (you guessed it, in memory). After all the efforts of securing the store, Larry was undermined by a third-party component. Had he simply ran a vulnerability scan via a man-in-the-middle testing agent against all services with egress (more importantly, so should have the email service vendor), this would have been detected well in advance.
