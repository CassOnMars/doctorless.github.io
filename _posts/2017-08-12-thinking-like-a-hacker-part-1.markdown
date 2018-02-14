---
layout: page
title: "Thinking Like A Hacker — Part I: Finding The Mindset"
permalink: /2017-08-12-thinking-like-a-hacker-part-1
---

In my [prior article](/2017-08-11-your-company-should-have-a-red-team), I encourage leadership at tech companies to adopt an official policy for permitting vulnerability research internally. The focus of this post is to discuss the mindset a team needs to get themselves into when engaging in pentesting. One of the first things I hear when talking to developers about vulnerability discovery is that they find it difficult to approach code (either their own or the code of others) with a malicious mindset. [Mencius would be proud](https://plato.stanford.edu/entries/mencius/#3), but that vastly tips the scale in favor of the bad guys, regardless of their philosophical motivations.

![mencius](https://cdn-images-1.medium.com/max/1600/1*HEgNXnh20V6p7x22QBH_yg.png "mencius")
“Your words, alas! would certainly lead all men on to reckon benevolence and righteousness to be calamities.”

So how does a benevolent mind approach security from an offensive perspective? Perhaps the best route is a little less Mencius, a little more Sun Tzu: know your enemy.

## Finding The Mindset Through Need: A Locked Front Door

Most people know to lock their door when they leave their house; a recognition that while most people are good, there are some who wish to do harm, and they will do so via the easiest avenue. A benevolent mind will often forget to check that the windows are also locked, until they find themselves in a situation where such thoughts need occur — i.e. locking oneself out of their home. To trick yourself into the hacker mentality, it may prove useful to put yourself in a desperate mind if nefarious is not a state you can enter.

Imagine a scenario where a theoretical developer (We’ll call him Larry) has accidentally deleted the configuration files, and thus the connection strings to the database. _“Larry, you did keep this info in a password safe, right?” “Oops.”_ His custom e-commerce site (note: I’m going to repeatedly pick on e-commerce with examples, as the architecture of these types of sites have a lot of moving pieces with very valuable information) remains happily chugging along, not being one to reload via config changes, but were it to tip over, he’d have a terrible outage. Larry has a copy of the config files for the development environment, which happens to closely mirror the production config, minus the connection strings. Larry also knows that the search feature of his application appends the search term to the query:

    // This hurt just as much to write as it does to read
    SqlDataAdapter searchAdapter = new SqlDataAdapter(
        "SELECT item_id, item_title, item_description FROM items " +
        "WHERE item_title LIKE '%" + 
        SearchText + "%' OR item_description LIKE '%" + SearchText +
        "%';", connection);

And lastly, because in this example Larry also isn’t very good at safe database configurations, the DB user in this connection string effectively has superuser privileges. Thus locked out of his house, Larry attempts to open one of your unlocked windows with a search parameter:

    foo%'; ALTER LOGIN ecomuser WITH PASSWORD = 'mynewpass'; --

Congratulations, theoretical Larry. You just performed a SQL injection, the number one threat on the [OWASP Top 10](https://www.owasp.org/index.php/Top_10_2013-Top_10). And if he could do it so easily, you can guarantee that someone else can, _and probably will._

![sadlarry](https://cdn-images-1.medium.com/max/1600/1*NIo3ePut7eovMFppTocBnw.png "sadlarry")
“I was able to restore my application, but…”

Now this above case was full of many other serious problems (not all inclusive, but in order of decreasing severity):

1. The database user used by the application in this example had superuser privileges. This should _never_ be the case. Full stop. The only applications which should have this kind of access are the tools which perform your database migrations, and even so they should also be limited to only having administrative control of the specific databases they _need_ control over. The policy of limiting access granted to the DB user to only what is required for the tasks performed by the application is a fantastic example of [The Principle of Least Privilege](https://en.wikipedia.org/wiki/Principle_of_least_privilege).
2. Production deployments are not an automated process. This was evidenced by theoretical Larry mistakenly deleting the config files. Other than opportunity for mistakes, a manual deployment process means the life of production servers are in the direct hands of people, who can also act with _malice_.
3. Similar to point 2, there isn’t an immutable, redundant infrastructure for deployments, evidenced by both the manual process for deployments and the fact that deleting a config file on a single server could undermine the application. I’m not so keen on moving everything into cloud services such as AWS (that is a post for another day), but even in locally-hosted environments, there are plenty of ways to have an immutable infrastructure (this is also a post for another day).

I certainly hope breaking into your real applications will employ more cunning. [But even the giants regularly fall](https://tools.cisco.com/security/center/content/CiscoSecurityAdvisory/cisco-sa-20170802-ucm). [Often](https://blog.sucuri.net/2017/05/sql-injection-vulnerability-joomla-3-7.html). Frequently, developers will avoid the potential for SQL injection by leveraging frameworks to abstract away database access, such as object-relational mapping frameworks or well-vetted data access classes, but turning a blind eye to the risk by assuming safety in third parties can be just as dangerous.

## Finding The Mindset Through Desperation: This Time An Actual Locked Door

Perhaps you _do_ make sure your windows are locked. Perhaps you have a phone-controlled door lock and you will _never_ leave home without your phone. Perhaps even, you wrote the door lock yourself because you can’t trust closed source lock providers ([and there’s precedent for that mistrust](https://www.wired.com/2013/08/kwikset-smarkey-lock-vulns/)). Since we’re in the land of theoretical scenarios, let me illustrate this “perfect” envisioning of such a creation:

![graph](https://cdn-images-1.medium.com/max/1600/1*eufnkks2_p0wlOgFYJVAxg.png "graph")
To this day, I still don’t understand how people are willing to connect something as crucial as their front door lock to the Internet, but then again most people don’t cover their laptop camera.

Your architecture is well-vetted. Your authentication and authorization logic is flawless. All SQL is properly parameterized, every action is logged and auditable. The only open port you have is for HTTPS, and your web client app is wholly separate from your lock API. Good for you! These are all smart choices. Your lock is full of awesome (configurable) features, comparable to commercial products:

* Bluetooth proximity sensor unlocks the door after extended departure
* Ability to query/set lock state from anywhere
* Scheduling to unlock/lock door at set times for expected visitors, such as pet sitters
* Scrollable message on LCD display informs lock state, can greet visitors
* Tiny speaker on lock can also project a welcome message to visitors when door is opened

All in all, a seriously solid project for someone going at it on their own. You even made it open source, and published the schematics. Your equally-geeky friends also thought it was awesome, and so now they all have one on their homes. It’s a runaway success.

One of your friends invites you over to their house for a party. You leave your laptop behind, knowing how wild these soirees can get. You open your Smart Lock app on your phone as you step out the door, and after authenticating with Touch ID, press the “LOCK” button, hearing a satisfying deadbolt click. Safe and sound. Time to party. As the night wears on with too many beers deep to leave room for wiser decisions, your friend decides to make margaritas, and so out comes the blender. You’re all far too gone already to notice that just before the ice went in, so too did your precariously perched phone. Add in some tequila, Cointreau, lime, and you’ve got yourself a Smashed Silicon Valley. Serve with umbrella toothpick.

![secret ingredient](https://cdn-images-1.medium.com/max/1600/1*8qouAWznomcw0a3mxt8NFw.png "secret ingredient")
‘The secret ingredient is “phone”.’

The next morning arrives, and both a hangover and dread sets in. Your phone is gone, and your laptop is home. Your friend tells you not to worry, just to log in through your web client you so carefully designed. Your friend means well, but doesn’t know you heed the advice of security professionals and use a password manager for everything — which was on your phone.

Your authentication/authorization logic is “flawless”; no way you’re going to exploit your way in as yourself. A locksmith in the area is going to cost you at least $300 for a lock so exotic, and you’ll probably still end up with damage to your door. The prospect of knocking out a window becomes appealing. You start to mull over the code in your head. “Maybe there’s something I’ve missed.” Now you’re starting to think like a hacker. Your friend lends you her laptop. Stepping backward through your logic: your goal is to open the lock. The lock is opened with a POST request to `/lock/{id}`, with a JSON object containing state:

    {
       "state": "unlocked",
       "message": "Unlocked",
       "opensound": "default"
    }

The API first verifies the authorization token, and if the token is not permitted to unlock the lock, then you are rejected with a 401. If only you could remember your authorization header. But then, you remembered something else that controls the lock — a schedule. The scheduler has a specific token that is authorized to unlock _any_ lock, but the API handler for scheduling state changes uses your authorization token to determine the right to change state…

    [HttpPut("/lock/{id}/schedules")]
    public async Task<IActionResult> AddScheduleToLock(
            string id,
            [FromBody]ScheduleLock scheduleLock) {
        if (!GetUser().HasRightsToLock(id)) {
            return Unauthorized();
        } else {
            scheduler.publish(new ScheduleLockMessage(
                id,
                scheduleLock));
            return Ok(scheduleLock);
        }
    }

At first glance, it doesn’t look promising. Let’s dive into the `ScheduleLock` class:

    public class ScheduleLock : Lock {
        public ScheduleLock Clone() {
            return (ScheduleLock)this.MemberwiseClone();
        }
        public DateTime ScheduleDate { get; set; }
        public Recurrence Recurrence { get; set; }
    }

You remember upon seeing this through your bleary eyes, that you wanted to future-proof the scheduler for any new features the lock may have, by simply extending the `Lock` class with a scheduled time. Let’s follow this through to the other side, the scheduler. It’s also the component you remember not being the most well-written. The scheduler has a subscriber to the pubsub queue you set up:

    public async Task HandleScheduleLockMessage(
            ScheduleLockMessage message) {
        if (DateTime.Compare(message.ScheduleLock.ScheduleDate, DateTime.Now) < 0) {
            Lock lock = (Lock) message.ScheduleLock;
            lock.Id = lock.Id ?? message.LockId;
            
            if (message.ScheduleLock.Recurrence != Recurrence.Never) {
                var newMessage = new ScheduleLockMessage(lock.Id,
                    message.ScheduleLock.Clone());
                switch (message.ScheduleLock.Recurrence) {
                    case Recurrence.Monthly:
                        newMessage.ScheduleLock.ScheduleDate = 
                            newMessage.ScheduleLock.ScheduleDate
                            .AddMonths(1);
                        break;
                    case Recurrence.Weekly:
                        newMessage.ScheduleLock.ScheduleDate =
                            newMessage.ScheduleLock.ScheduleDate
                            .AddDays(7);
                        break;
                    case Recurrence.Daily:
                        newMessage.ScheduleLock.ScheduleDate =
                            newMessage.ScheduleLock.ScheduleDate
                            .AddDays(1);
                        break;
                }
                scheduler.publish(newMessage);
            }
            
            DB.Locks.Update(lock);
        } else {
            scheduler.publish(message); // just re-enqueue it
        }
    }

After mulling it over for a few moments, you notice you are assigning the Id property of the `Lock` if it isn’t _already_ set. Your `AddScheduleToLock` API handler simply deserializes whatever properties of `ScheduleLock` are provided to it, and the `Id` value on the JSON object is _never checked_. Uh oh.

In OWASP terms, this falls under #4 of the Top 10: Insecure Direct Object References, by virtue of a _Mass Assignment_ vulnerability.

Creating a test account, you PUT a schedule on your test lock’s URI, using your home lock’s Id (which you accidentally committed early on in some test code, but later removed) in the JSON:

    $ curl -X PUT -H "Authorization: Token WW91IGRpZG4ndCB0aGluayBJJ2QgYWN0dWFsbHkgcHV0IGEgcmVhbCB2YWx1ZSBoZXJlLCBkaWQgeW91Pw==" -d '{ "state": "unlocked", "id": "e5ea783d-b32a-4716-8001-89d7e035c2df", "scheduledate": "2017-08-12T08:00:00Z", "recurrence": "Never" }' https://api.example.com/v1/lock/4ee62a78-0970-4a2f-bd68-0a678944bb6a/schedules -v
    *   Trying api.example.com...
    * TCP_NODELAY set
    * Connected to api.example.com (0.0.0.0) port 443 (#0)
    * ALPN, offering http/1.1
    * Cipher selection: ALL:!EXPORT:!EXPORT40:!EXPORT56:!aNULL:!LOW:!RC4:@STRENGTH
    * successfully set certificate verify locations:
    *   CAfile: /opt/local/share/curl/curl-ca-bundle.crt
    CApath: none
    * TLSv1.2 (OUT), TLS header, Certificate Status (22):
    * TLSv1.2 (OUT), TLS handshake, Client hello (1):
    * TLSv1.2 (IN), TLS handshake, Server hello (2):
    * TLSv1.2 (IN), TLS handshake, Certificate (11):
    * TLSv1.2 (IN), TLS handshake, Server key exchange (12):
    * TLSv1.2 (IN), TLS handshake, Server finished (14):
    * TLSv1.2 (OUT), TLS handshake, Client key exchange (16):
    * TLSv1.2 (OUT), TLS change cipher, Client hello (1):
    * TLSv1.2 (OUT), TLS handshake, Finished (20):
    * TLSv1.2 (IN), TLS change cipher, Client hello (1):
    * TLSv1.2 (IN), TLS handshake, Finished (20):
    * SSL connection using TLSv1.2 / ECDHE-ECDSA-AES128-GCM-SHA256
    * ALPN, server accepted to use http/1.1
    * Server certificate:
    *  subject: C=US; ST=Somewhere; L=Somewhere View; O=Example Inc; CN=*.example.com
    *  start date: Aug  2 19:27:21 2017 GMT
    *  expire date: Oct 25 19:23:00 2017 GMT
    *  subjectAltName: host "example.com" matched cert's "example.com"
    *  issuer: C=US; O=Example Inc; CN=Example Internet Authority G2
    *  SSL certificate verify ok.
    > PUT /v1/lock/4ee62a78-0970-4a2f-bd68-0a678944bb6a/schedules HTTP/1.1
    > Host: api.example.com
    > User-Agent: curl/7.53.1
    > Accept: */*
    > Content-Length: 132
    > Content-Type: application/json
    >
    * upload completely sent off: 132 out of 132 bytes
    < HTTP/1.1 200 OK
    < Connection: close
    < Content-type: application/json
    <
    * Closing connection 0
    {"state":"unlocked","id":"e5ea783d-b32a-4716-8001-89d7e035c2df", "scheduledate":"2017-08-12T08:00:00Z","recurrence":"Never","message":null,"opensound":null}

Talk about mixed emotions.

---

In Part II, I will discuss the mindset from the perspective of side-channel attacks. 
