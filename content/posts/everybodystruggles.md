---
title: "Mire/Perspire/Despair/Inquire/Aspire"
date: 2020-12-22T20:39:22+05:30
draft: false
---

___'When jarred, unavoidably, by circumstance, revert at once to yourself, and don’t lose the rhythm more than you can help. You’ll have a better group of harmony if you keep on going back to it.' — Marcus Aurelius,___
-- quoth Sumera to herself after struggling for an entire week on how to start this piece of writing, and finally beginning it with a famous quote.
Everybody struggles (even with something like writing about struggles), and here is

### Lesson 1:

> *You don't have to find a perfect solution by yourself everytime.  Take the next step, give yourself time and keep coming back to it, improving it as you are helped by the community and you come up with better ideas.*

In this case, what I was really struggling with is the idea of a perfect beginning and that kept blocking me from actually writing the rest of the article:

_What if nobody feels like reading it past the first paragraph? If the article is entirely unrelatable? Why am I struggling even when I know the rest of the community is wonderful and will not judge my writing skills?_

In hindsight I should have asked for help from peers and tech bloggers- but it never occurred to me to ask for writing advice from all the tech communities I am part of - because I have been (quite blindly) seeing both disciplines as unrelated. Just like the time I got stuck on an Ocaml issue and realised I don't know anyone who knows Ocaml (apart from my mentor at the time) before a friend reminded me what I do know is Reddit and could always ask there.
This time, thankfully a dead Roman emperor turned up on Google to help, and in the subsequent process of writing, I have (just) discovered,

### **Lesson 2 (and reminder to self):**

> *Keep a list of `where to ask` resources handy lest you forget that you have access to brilliant communities and there are people you can always ask for help without fear of judgement*

Now, before I digress again, allow me to tell you how I struggled with the initial setup of the VKMS system. As a relatively new driver in kernel age, very little has been documented for the VKMS driver outside of the patches and blog posts by previous contributors. So, my mentor, Daniel asked to me add basic setup documentation for my first patch.  We decided to include compiling, loading and basic testing with IGT as my first patch.

I had already compiled and loaded VKMS so I only had to finish testing.  When I ran the tests though, I got an error saying VKMS is not the master device. I asked in the irc, and people pointed out that the flag I was using- `IGT_FORCE_DRIVER`-  has been deprecated so maybe that is the reason. They suggested another variable which my other members confirmed worked so I tried that. Still, no show. That is when Melissa pointed out that I should be running the tests in text mode, with GUI disabled.  Since I was not doing that, the X server stayed as master and hence all my tests were failing.

Now I should have just asked how best to switch to text mode. However, under the naive impression that it is simple and there would be one way to do it, I used,

   ```
   sudo systemctl disable gdm
   ```

Sure enough it worked. GUI turned off, and I time-travelled, facing a machine with only the console to interact with.
The tests succeeded, and I kept a log. All this time, I was super excited and marvelled at how early programmers accomplished so much with computers without any help from a GUI.

Then, it was time to switch back to GUI. The source I was referring to asked to use
   ```
   sudo systemctl enable gdm
   ```
to do this. **Oops**. Error. It won't switch back. I restarted. I tried various tutorials. I got overwhelmed by information about systemctl and gdm and display managers. Sacrificed a meal. Burned prayers. I tried more forums and their instructions. All to no avail.

At which point I finally figured out what had happned.  `display manager` is a symbolic link that usually points to `gdm`(gnome display manager). The link however got destroyed when I did what I did and now when I was trying to enable it back on, `systemctl` was unable to find the actual `gdm` and link to it.  So all I needed to do was set the link again.  Thankfully, by this time I had already asked Melissa for help. She found a link that told me how exactly to create the symlink between display manager and gdm and  voila, I had my display back! Thereafter Melissa told me to use `systemctl isolate` to switch to and from text mode, and to try and not touch gdm.  That has worked like a charm.  That brings me to:

### **Lesson 3:**

> *It is okay to keep trying, but reach out to people as soon as possible. Help will arrive*

Speaking of struggle, I have another anecdote to share. So, during the contribution period for Outreachy, I was supposed to send a few patchsets. By then, I had been contributing for a few months already. And had prior experience sending patchsets. So I was not really worried at all - I found  some relevant issues in `staging-testing` I could fix and sent the patchset. Oops.

I checked the mailing list to find that the patchset had gotten scatterred across different lists-  a handful of recipients not getting a cover letter and some not getting the patches. Since there were a lot of maintainers across the three patches I was making, I had tried to send each patch to only the maintainers for that individual patch and in the process had skipped adding everyone to the cover letter. No wonder. I rectified my mistake and resent the patches. Then I decided to send another patchset. Oops again. Same thing happened except no idea how I messed up this time and to this day, I still am not sure.

It wasn't that I didn't know how to send a patchset- it was that everytime I sent one with specifcally Outreachy in mind- I kept making one stupid mistake or the other while the other patchsets I sent before or after or around the same time went perfectly smoothly.  I felt so stupid and annoyed at myself- imagine knowing something and still failing miserably at it! So here goes,

### **Lesson 4:**

> *Be compassionate towards yourself when you struggle to do well, just as you would for others who make mistakes.*

Now, well into the third week of my internship, here is what I am currently struggling with-  trying to ascertain why my mouse starts behaving as if it belonged to Windows 2000 everytime I load the VKMS driver with gui on, the behaviour exacerbated as my battery depletes. With all the cursor tests passing too! Notably, this started after I applied a year old patch and applied to the latest 3 days old dri-devel kernel but then I undid that and the error still persisted, so I am quite baffled. Thankfully, I don't yet need to load VKMS with display on, and hopefully, I will figure it out in a few days.

### **Lesson 5:**

> *Sometimes, you may never figure out the problem in limited time. Move on. No point ending up as Sisyphus.*

Obviously, as I delve deeper into the system- I will struggle a lot more. The silver lining is that all the struggling so far has taught me to reach out and stop worrying excessively, and I am glad that I have mostly overcome my fear of public forums. Looking forward to documenting my further struggles!
