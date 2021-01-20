---
title: "Unfolding"
date: 2021-01-17T20:39:22+05:30
draft: false
---

It has already been six weeks since my internship started and I am much less of a
clueless beginner than I was when I had first applied for this project. But still a little clueless about a couple of things which is why this post is after such a long gap.

This one is mostly going to be a midpoint progress update post. And pretty short because there hasn't been a lot of progress (at least not as much as I would like, given the number of patches). My mentors assure me it is okay and this happens so I will try to not blame myself too much.

![not the end of the world](https://media.giphy.com/media/3orifc9rWOO93wPGRW/giphy.gif)

Now, as far as dri-devel is concerned, most internship projects are of the plan-it-yourself 
kind. Applicants are expected to come up with a project plan after discussing potential project ideas from the [TODO](https://www.kernel.org/doc/html/latest/gpu/todo.html) list with maintainers. I decided to go with a VKMS based project since it was did not have any particular hardware dependencies. Also, it had beginner level tasks which were suitable for me as I had barely written any kernel driver code earlier.

After consulting with Daniel and Melissa, we decided on working to improve the VKMS driver by adding some features as module options and exposing the driver through configfs. Here's mostly what the plan looked like initially:

	- WEEK 1-2: Introduce configfs | Include `enable_cursor` as a `config_item`
	- WEEK 3-5: Implement Commitable Items
	- WEEK 6-8: Support for Vblank and Writeback operations
	- WEEK 9-10: Polish `virtual_hw/vblank less` mode
	- WEEK 11-12: Integrate all modules with configfs

Here's what I have done from that plan so far:

	- WEEK 1-6: Expose cursor and writeback as module options.

So, what happened?

Once I took the first few weeks to acquaint myself with the project,
get my setup in place, and send in a [tiny docs patch](https://lkml.org/lkml/2020/12/9/951), we decided to slightly modify the
plan a bit. Instead of first doing the configfs part and then module options, Daniel suggested we do stuff in parallel. We

- start off with some prep work for adding configfs
- alternate this with adding module options

Thanks to this early advice, [cursor](https://lkml.org/lkml/2021/1/10/107) and [writeback](https://lkml.org/lkml/2021/1/10/108) have already been implemented as module options. They were low-hanging fruits so all that had to be done was expose them as modules and that got done soon enough. Which surprised me because I thought I would have a lot of trouble with this part of the project. Naive as I was, I had no clue what adding module options entailed and how I would go adding these new features and I expected this to be really tough.

Of course I was entirely wrong. Soon enough into the project I realised configfs is nowhere as simple as I had assumed it to be. I have spent a lot of time, even till now, only to understand why it does what it does and even that is complicated as hell. I am beginning to understand the api a bit, so just started on the how it does whatever it does part. 

Now, what the initial plan required me to do is the next step which is to _use the api succesfully_. I will never understand why I assumed that to be an one-dimensional 3 stepped task and that I would sail through it with flying pixels. 

![ignorance is bliss](https://media.giphy.com/media/e67jW3CzzzWus/giphy.gif)

The best part, however, was that despite knowing the plan is over-ambitious and it may go wrong, my mentors allowed me the space to explore and figure things out on my own without hurrying me to change the plan. As a result, I got time to analyze where exactly I went wrong, and come up with a better strategy without being rushed or panicked.

I had assumed adding configfs would be just getting this to happen in a step:

	configfs
	└── vkms
    	    ├── vkms0
    	    │   ├── enable_cursor
    	    │   └── enable_writeback
    	    └── vkms1
        	├── enable_cursor
        	└── enable_writeback


And I had also assumed I would understand all about configfs in a few weeks. Daniel explained to me that I had put nearly twice the work in my plan as what's expected for an internship. 'Implement configfs` comprises much more than what my plan assumed. Here's the steps I had overseen:

	- multiple vkms instances
	- locking 
	- behaviour of module options
	- error handling
	- integrating every vkms part
	- implementing commitable items

So we recalibrated. We decided to just reduce our goals and try to just get vkms devices to show up at the configfs mount point and mark the remaining parts as stretch goals. So, mostly, just find a way to register and unregister the subsystem through configfs, and even that would require a lot of experimentation.
With that, the new plan mostly looks like this:

	- Implement vblank and virtual_hw as module options
	- Just expose vkms device at the configfs mount point
	
Between the previous and current plan, here are what I have learned so far:
	
	1. How drm and kms and vkms fit together
	2. IGT tests
	3. Using IRC (this is big, okay? been trying to figure this out since 2017)
	4. Text only mode 
	5. Scouring the kernel for docs/references
	6. Experimenting without being scared
	7. Driver modules
	8. the point of VKMS
	9. the point of configfs
	10. the point of vblank

Mysteries still abound plenty tho,

	1. How to actually befriend configfs?
	2. Implementing locking in the kernel
	3. Steps to implement vblankless mode
	4. Connectors
	5. Framebuffers

These areas, except configfs, are areas that I haven't really explored in detail yet. Hence, still in the process of demystifying.

In retrospect, I am a tad glad this happened. I learned some nifty time management and project planning skills on the way. Here's to looking forward to achieving the rest of my goals on time.