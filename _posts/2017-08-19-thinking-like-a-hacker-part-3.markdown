---
layout: default
title: "Thinking Like A Hacker — Part III: At Smart Lock, Your Voice Is Your Password"
permalink: /2017-08-19-thinking-like-a-hacker-part-3
---

In [Part I](/2017-08-12-thinking-like-a-hacker-part-1), I cover an approach to finding the hacking mindset through looking at your own code. In [Part II](/2017-08-14-thinking-like-a-hacker-part-2), I approach this discovery through an analysis of an active hacking attempt. Disclaimer: This story (as well as prior stories) is entirely fictional. While I may refer to publicized vulnerabilities in passing, the “discovered” vulnerabilities within the story do not correspond to any actual product.

---

Return to the Smart Lock adventures. Theoretical you, having just broken into your home by virtue of exploiting your own product, realize you have some work to do. Of course, just fixing bugs won’t do. You’ve got thousands of stars on GitHub and dozens of pull requests on your issue tracker to review! One pull request stands out as the most interesting next feature: voice mail.

![hey](https://cdn-images-1.medium.com/max/1600/1*frytsUpkN8Dfox0xEcHEdA.png "hey")
On second thought…

> Feature description: The ability for a visitor to leave a message, either by microphone on the lock, or through the app when in proximity to send a voice message by bluetooth. Messages can be reviewed either on the lock, or through your account on the app.

Neat.

Concerned by your own break-in, you decide it wise to consider all possibilities, no matter how esoteric, that may lead this feature back into your home. A few things spark your imagination:

1. Too long of a message could run your lock’s local storage out of space. _Need to enforce storage limits, or just stream it from the API._
2. In order to leave a message through the app, the lock will need to broadcast an identifier to local bluetooth devices to associate the message with the lock. _Broadcasting the ID of the lock shouldn’t (in theory) lead to compromise, but you already know how well that went for you. Better to have the lock generate a new ID to use to store the message._
3. While the only way you can get a message to the lock is by proximity, there’s nothing stopping you from getting the ID, then sending whatever you want as a file manually. _Got to check the file type of the submitted data. More importantly, it needs to actually check the file, not just the file extension._

Other than those concerns, it seems like a solid PR. You merge it into a new feature branch so you can add your changes, and spin up a test environment.

![dos boot](https://cdn-images-1.medium.com/max/1600/1*3JkbVw0x0dirJBgVMooJZA.png "dos boot")

So far, so good. You fire up a test copy of your phone app, and leave a voicemail. Playback works successfully. Extra neat. The prospect of sound playback from user-submitted files still gets to you. You’ve seen the lesson learned by others in the industry: always assume user-provided data is hostile. Heck, you _were_ the hostile user data. Time to get creative.

Buffer overflow exploits are a known type of vulnerability to watch out for. You’ve heard of them in practice, you even jailbroke your phone with one, but you’ve never really employed one of your own crafting before. You’ve vetted everything else about the PR, but the audio playback is being performed through a third-party (albeit open source) application.

Yet-another Open-source Library for Orchestrating Sound, as WAV or Another Genre — or YOLOSWAG for short (please never name your project this), being the most popular library for, well, orchestrating sound, is the component of choice. Being as popular as it is, it supports a variety of formats, including some rather obscure codecs you’ve never heard of. With this many independently-contributed codecs, there’s got to be one that wasn’t well-written.

![yolo](https://cdn-images-1.medium.com/max/1600/0*44dxKxmKkMyhy1n8.jpg "yolo")
I mean, with a name like that, how could it not be secure?

In open source projects and closed source projects alike, developers put a lot of their attention towards features which see the most use. This has a correlation with the areas of attention given to security from these same developers. It’s not a criticism; a balance always has to be struck with limited resources, but it can’t be forgotten, that while developers are putting most of their attention towards improving features and creating value-adds, those wanting to break in will have the easiest time usually in the components that did not receive the same time dedication.

Looking through the YOLOSWAG commit history, you see that one particular format, Audio to the Eleventh Power (or AAAAAAAAAAA, for short) hasn’t seen any changes since it’s first commit, which was provided by a relatively new, infrequently-active committer. A quick review of the codec’s README indicates the format was made as a senior final project for their CS degree. This _compelling_ backstory granted an approval by the project maintainers to merge it into the master branch.

In audio file formats, there can be many layers of complexity. Some file formats serve as containers for audio data encoded through a variety of codecs (AIFF and WAV come to mind). Some formats are singular in nature; limited to a specific codec, perhaps lossless, lossy, or no compression measures involved. The greater the complexity, the more possibilities for mistakes, and the harder it becomes for automated pentest tools to find them, if any are being used at all.

AAAAAAAAAAA, on the other hand, is pretty simple. Lower level languages aren’t your forte, but the simplicity makes it a pretty easy read.

    static int aaaaaaaaaaa_decode_frame(YOLOSWAGContext* context, unsigned char* data, YOLOSWAGPacket* packet) {
        unsigned int frame_size = 0;
        char frame[4096] = memset(; // each frame can only be 4096 bytes.
        memcpy(&frame_size, data, 4); // copy the first four bytes from the data
        
        data += 4; // advance the data four bytes
        memcpy(frame, data, frame_size); // copy the data into frame <<<<<<
        // ...
        // decoding logic on frame occurs here, then creation and assignment of decoded PCM data to the packet.
        // ...
        return 0;
    }

“Wait, did that just do what I think it did?”

Yeah, you got yourself an opportunity for a buffer overflow. The author of AAAAAAAAAAA didn’t bother to check if `frame_size` was greater than the actual amount of space allotted to `frame`.

Of course, there’s a lot of code in the way of getting there. No harm in trying though. You write an audio file in the AAAAAAAAAAA format, using the conversion tools helpfully provided by the YOLOSWAG maintainers. The sound of Tim Allen’s [AUUUGH](https://www.youtube.com/watch?v=KnsiZOJjfUg) seems suitable. A test run verifies that the sounds of the 90s are still alive today, even in an audio format written by someone who was too young to have seen it. Time to fire up a hex editor.

Changing the `frame_size` value in the last segment of the file to 5000 should do nicely. Adding some extra garbage data at the end to pad the file to the size, you now have your first test.

![bodes well](https://cdn-images-1.medium.com/max/1600/1*7e8CAbu8sZJkbpmWaSMx2g.png "bodes well")
This bodes well.

Sure enough, not only did it segfault, you got no word of a stack smashing warning. A little more tweaking to your audio file so you can rearrange the stack to return to a libc call to `write` to your serial device (and unlock your lock), and you get [this wonderful outcome on your test lock](https://streamable.com/s/vuxvg/sqacun). 

After disclosing your findings to the YOLOSWAG team, you decide it best to limit file support to MP3, and execute the player process as a user with severely limited rights.

AAAAAAAAAAA, indeed.
