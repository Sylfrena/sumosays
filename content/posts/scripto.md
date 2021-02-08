---
title: "Extra keystrokes and Jugaad"
date: 2021-02-07T20:39:22+05:30
draft: false
---

This post is going to be a tiny piece about a tiny script that I wrote
out of sheer lazyness. Currently, I am trying to implement a vblank-less
mode for the VKMS driver. Since the VKMS driver aims to simulate hardware
graphics, vertical blanks are also simulated using an hrtimer interrupt.
And this code is tested using igt-gpu-tools.

However, what if we want to simluate virtual hardware? That's where the vblank-less
mode comes in. The essential idea is to bypass the interrupts and directly perform
the composition work from the atomic hook for the driver. To get this done,
I not only need to test my code at every iteration, but also trace the function call
chain to see what's going on under the hood while my tests are running.

In order to trace the function call using ftrace, here's roughly what I need to do (as root):

    1. Make sure the trace log file is empty
    2. Switch on ftrace
    3. Start test
    4. Log the test results into a file.
       (name the file meaningfully)
    5. Stop test
    6. Switch off ftrace
    7. Log ftrace results into a meaningfully named file.
    8. Repeat steps 1-7 for the next test.

Sounds like less work? It actually is pretty less unless you have an RSI or arthritis and then every keystroke makes a difference. Plus when would me taking that shell scripting lab for an entire semester come in handy? As my mentor says, [everyone makes a script](https://melissawen.github.io/blog/2020/05/20/community-bounding), so I too, set out to do the same.

Here's what it looks like: 

```
#!/bin/bash

function get_time() {
	date +"%m%d%H%M%S"
}

function run_igt_tests() {
	name=$1
	prefix=$2

	TEST="./build/tests/$name --device "sys:/sys/devices/platform/vkms""

	test_name="${prefix}.test"

	$TEST > ${test_name}

	echo "test file name is $test_name"
}

function run_trace() {
	name=$1
	prefix=$2

	trace_name="${prefix}.trace"

	echo ""  > /sys/kernel/debug/tracing/trace
	echo 1 > /sys/kernel/debug/tracing/tracing_on

	run_igt_tests "$name" "$prefix"

	echo 0 > /sys/kernel/debug/tracing/tracing_on
	cat /sys/kernel/debug/tracing/trace > ${trace_name}

	echo "trace filename is $trace_name"
}

function test_things() {
	declare -a arr=("kms_vblank" "kms_writeback" "kms_cursor_crc")
	for i in "${arr[@]}"
	do
		now=$(get_time)
		prefix="${i}_${now}"
		run_trace "$i" "$prefix"
	done
}

```
To run the script, all you need to do is source it and call `test_things()`. To add more tests, just modify the array in `test_things()`.

I think what took me the longest with this smol script is to get the file names correct. I wanted each name to reflect the type(test/trace), the time, and the test name. Often what would run on my usual zsh simply wouldn't run when I ran it as root in bash.

A caveat in the script is that it needs to be ran from the `igt-gpu-tools` folder root. I would like to change this in the future and also make sure that the test and trace logs are stored in a separate folder rather than just in the igt folder root.

For now though, this _jugaad_[^1] works pretty well for me so I think it'll be a while before I update the script.

#### *Footnotes*
  
[^1]: _Jugaad: low effort hack/workaround_
