---
layout: default
title: "Your Company Should Have A Red Team"
permalink: /2017-08-11-your-company-should-have-a-red-team
---

![red team](https://cdn-images-1.medium.com/max/1600/1*uSGYOkkgQ-XVKfdXVOPQjw.png "red team")

In the parlance of the Intelligence Community (IC), there exists the concept of a “Red Team” — an oppositional yet internal force which has the sole purpose of finding weaknesses, in order to assess risk and improve effectiveness at handling risk. In the corporate world, this term is more narrow, in that it typically applies to a team tasked with finding vulnerabilities, both in the software produced within, and the surrounding infrastructure (physical and digital). If you don’t already have one, you should seriously consider starting one. If it is too much to ask to have resources dedicated to this full time, then it should at least be something that an internal team is performing part time, preferably as a blend of members from different engineering teams. The resource cost of not having one may far outweigh the possible financial, reputational, and legal costs — an ounce of prevention, etc. While I write this article with an aim towards tech companies, this isn’t something to be ignored by non-tech industries; even a theoretical paper-only clinic need worry about social engineering, for example.

## Why Should I Care?

As time marches on, hardware gets cheaper, and the Internet grows more deeply connected. Cyberattacks once considered sophisticated a decade ago are now a normal, automated component of an attacker’s arsenal. Today, fuzz testing serves as the method of choice for many bug bounty hunters. Tomorrow, this will likely be just another step in the automated scanners crawling the web on behalf of criminals in attacker-friendly nations. Having a team internally deploying the same tactics without having to worry about detection¹ gives you a greater peace of mind in ensuring the safety of your business.

Detection concerns aside, a red team also has an advantage that attackers do not always have. Information about your infrastructure: the network map, the equipment used, logs, and especially the _source code and application infrastructure_ all make a potential hacker’s job exponentially easier — and your team has it from the start.

Don’t read into the above as an implicit approval of security through obscurity. There is little benefit to insider knowledge overall, and assumptions of safety guarantees can make teams more prone to making very costly mistakes. This applies in multiple levels. Many large businesses have internal supporting applications, where the assumption of the application remaining internal has lead to [actual data breaches](http://www.zdnet.com/article/anatomy-of-the-target-data-breach-missed-opportunities-and-lessons-learned/).

## How Do I Assemble A Red Team?

If you don’t already have an active security or risk management team that can take on this role, you can certainly source from within. There are many engineers that are naturally curious, however they may feel fear of retribution either from other team members or their superiors if there is not a written policy allowing this. Conversely, you would not want to grant your employees the right to start digging around wherever they’d like. Having a clear call to action for a pentest team to assemble, with some reasonable guidelines on what constitutes acceptable behavior can go a long way.

The limits to allowable discovery should be reasonably soft, as long as proper disclosure is made to the right individuals who can record the incident and rectify it. In some sense, this policy should resemble the rules surrounding bug bounty programs many businesses offer. For starters:

1. Define the areas of most interest, ranked in priority. This will likely start at your most popular software, down to the components and equipment (if applicable) your applications rely on.
2. Define what is out of scope. Production servers with customer data would likely fall under this, as would internal information not related to the product your company offers (like HR records, for example).
3. Devise a secure disclosure policy. This can be as simple as walking over to the affected team to let them know and demo a proof of concept, to encrypted reports with a decryption key shared with the affected team. In larger companies, it can be riskier to use ticketing systems with all-employee access, as an unscrupulous employee can walk out the door armed with this information.
4. Optionally, offer rewards based on the severity of the vulnerability.

There is naturally going to be a lot of variance for your implementation, as every company has their own distinct no-go zones, and sometimes those can even have overlap with their own product if they are [dogfooding](https://newrepublic.com/article/115349/dogfooding-tech-slang-working-out-glitches) (and if you can, you should!).

Unlike WarGames, the only winning move _*is*_ to play.

---

[1] Bear in mind, there are considerations to make if your infrastructure is hosted in third party services; most require disclosure upfront about automated scans and penetration testing, and that you do so at a time scheduled in advance.
