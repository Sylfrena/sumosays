---
title: "So long and thanks for all the fish, Outreachy!"
date: 2021-03-06T20:39:22+05:30
draft: false
---

The best experiences often pass by in a flash- mostly because we are so engrossed in giving our best and enjoying these moments that we forget to measure them in time. Of course, that (plus the temporal lopsidedness in this pandemic) is primarily how I ended up thinking February has thirty one days and another week left till my Outreachy internship ends. Guess who was caught off guard when the end _marched_ in so abruptly?

But now that my internship is over, here goes a final report of sorts. During my internship, I sent the following patches:

For drm/vkms:


- [drm/vkms: Add crtc atomic helper functions for virtual mode](https://patchwork.freedesktop.org/patch/422746/)
- [drm/vkms: Add support for virtual hardware mode](https://patchwork.freedesktop.org/patch/422745/)
- [drm/vblank: Fix typo in docs](https://patchwork.freedesktop.org/patch/414211/)
- [drm/vkms: Add information about module options](https://patchwork.freedesktop.org/patch/413452/)
- [drm/vkms: Add support for writeback module](https://patchwork.freedesktop.org/patch/413125/)
- [drm/vkms: Add vkms_config type](https://patchwork.freedesktop.org/patch/413450/)
- [drm/vkms: Add setup and testing information](https://patchwork.freedesktop.org/patch/406523/)

For igt-gpu-tools:

- [tests/kms_cursor_crc: Skip vblank waits for virtual_hw mode](https://lists.freedesktop.org/archives/igt-dev/2021-February/029401.html)
- [tests/kms_vblank.c: Skip vblank tests for virtual_hw mode](https://lists.freedesktop.org/archives/igt-dev/2021-February/029355.html)
- [tests/kms_lease: use igt_wait_for_vblank()](https://lists.freedesktop.org/archives/igt-dev/2021-February/029064.html)
- [lib/igt_kms: Update documentation for __igt_vblank_wait()](https://lists.freedesktop.org/archives/igt-dev/2021-February/029068.html)
- [lib/igt_kms: Decouple ioctl call logic for vblank wait](https://lists.freedesktop.org/archives/igt-dev/2021-February/029067.html)


The first half of my internship mostly comprised minor patches that helped me get familiar with the
VKMS driver. While sending the first patch, I learnt how to setup the VKMS driver and run tests with
igt-gpu-tools. After this, I sent patches to add `cursor` and `writeback` features as module options
for VKMS. Even though these patches were low hanging fruits they helped a lot not only with understanding
various features and design of the VKMS driver but also with boosting my confidence. In the second half of my internship,
we finally started working on adding virtual hardware support to VKMS- this would occupy the rest of my internship.

The VKMS driver is supposed to aid the testing and development of graphics drivers without having to use the actual
graphics hardware. One of the features we were trying to add is emulation of virtual hardware- this would essentialy mean
a mode where vertical blank interrupts are not used. We decided to implement this by bypassing the vertical blanking 
events. What I thought would mostly be a matter of a small conditional rewiring of control flow blew up into
a wholly unanticipated scenario with no clear solution in sight.

On the igt-tests side, this meant a tiny refactor in how the vblank code is tested. We extracted the ioctl call into
a helper function which we called in the test functions igt_wait_for_vblank() and igt_wait_for_vblank_count(). This would
prove helpful for skipping the igt vblank waits later.

Next we decided to change things on the vkms side: Call the composer directly from the atomic hook while bypassing the vblank
related functions. This involvd two steps:

 1. A new composer function without all the locking that was previously required for racing with vblank interrupts
 2. Calling this new function from the atomic hook.

The first one was not very tough- just remove the lock constructs. The second was tough- how do I determine the control flow, given I had very little idea of atomic mode setting ? If we were skipping the existing `vkms_composer_worker()` function, what's the best place to call the new `vkms_crtc_composer()` function? Being able to figure this out meant learning ftrace and dmesg logs, asking lots of questions, a random sprinkling of printks, and multiple attemps writing a blog explaining this.[^1] Once I figured this out with help from my mentors, it was mostly a matter of cleaning up, turning this into a module option and a few if-else statements.

That should have solved it, right? It didn't- all tests were failing. Vblank failures. We skipped all vblank tests. Still failing.
Then we realised that the problem was actually crc- we had changed the composer function to be oneshot but crc calculations were scattered throughout and relied on vblanking events to work correctly. After a [discussion on the mailing list](https://patchwork.kernel.org/project/dri-devel/patch/20210224105509.yzdimgbu2jwe3auf@adolin/) on how actual virtual hardware does this, we realised it will not be a minor refactor but lots of rewiring on the kernel side, and the tests as well. So the final patch I sent for my internship was the RFC patch for adding virtual hardware support to VKMS which is still under review.

There is a long way to go:

    1. Implementing crc as a oneshot operation
    2. Implementing configfs

I would like to continue working on this after Outreachy as well, although at a slower pace because I also need to job hunt, hehe.
Although I could not complete everything I had in mind during the internship, I have come a long way from the beginner I was before I started Outreachy to the beginner I am now.

### **There is a lot I learned in just these three months:**

1. Software can be both extremely tough and rewarding.
2. Planning is __really__ important, and that is not just making a time schedule with parts of the project. Research into potential roadblocks and allocating space for them is essential as well.
3. Working as part of an inclusive community makes a lot of difference- both in terms of motivation for and quality of work.
4. The enthusiasm in dri-devel is nothing short of amazing- I think the only time there was a lot of silence was during the holidays.
5. Sometimes schedules and tests don't work out the way they should. It is okay, just make sure to ask for help if you are stuck.
6. I finally can _confidently_ say I know how to write C and can write tests. Although it's been five years since I first wrote C and have even done projects and algo classes in C[^2], I never felt like _Oh, I know C_. That has changed to some extent.
6. Asking questions! This one has taken a lot of courage battling imposter syndrome. It is easy to feel like you are asking too soon or are wasting your mentors' time asking questions but as a fellow Outreachy intern, [Juliana points out](https://jufajardiniwordpress.com/2020/12/18/oh-the-struggles-part-1/), the community, especially the mentors, want to help which is they signed up for Outreachy. Although I still feel very shy, I can at least ask questions on forums so yayy.
7. I had an impression that Outreachy is majorly about building software- now I realise it is more about building a software community which is diverse, inclusive and welcoming.


### **Here are a few notes for self:**


1. I felt documentation to be much tougher than the coding part- often writing commit messages took me more time than the patches themselves. I need to write more documentation patches to improve this.
2. My general communication skills need to improve- I sometimes change context too abruptly or phrase my doubts wrong; that needs to change. I also feel I could have interacted more with the community, not sure how tho.
3. I definitely need to be less slower. Again, something I don't know how to change that except just work harder and hope XD
4. Sleep is important. Taking timely breaks from the screen before screen nausea sets in is also important. Plus this often helps in arriving at solutions faster.
5. All of it has been explained in the docs. But it may take you forever to find it, so just ask if you can't.

Throughout my Outreachy internship, not once was I made to feel unwelcome or incapable. Although I am very junior, the dri-devel community is very inclusive, encourages discussions and disagreements, and never made me feel like an outsider. I am very grateful to my mentors, Daniel and Melissa for their willingness to help, and for just being super nice. Thanks also to Julia Lawall who encouraged me to apply for Outreachy! A huge shoutout to Sage Sharp, Anna e só and my fellow Outreachy interns for always being so inspiring throughout and building safe spaces for us all.

As for me, I am surprised to have made this far. I kept thinking my mentors chose the wrong intern for the job and would finally realise it sometime during the internship, but they both think the internship was a success so I will try to believe them on that.
I finally believe I belong in tech, and that's pretty huge, so, I am awfully grateful that Outreachy happened. I can call myself an Outreachy intern now!

#### *Footnotes*

[^1]: Of course they were only attempts- the complete post is still manifesting. But here's a great article series by Daniel Vetter explaining atomic: https://lwn.net/Articles/653071/
[^2]: Unsolicited advice: Leetcode kinda algo problems in C is a terrible idea btw, you lose time implementing basic things which you can instead use to solve the problem.