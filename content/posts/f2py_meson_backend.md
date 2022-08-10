---
title: "Week 5: Finishing with F2PY's new frontend"
date: 2022-06-15T21:34:26+05:30
draft: false
slug : ""
authors : ["Namami Shanker"]
tags : ["F2PY", "argparse", "cli"]
categories : ["F2PY", "Programming"]
externalLink : ""
series : ["GSoC'22"]
---

# Week 5: Finishing with F2PY's new frontend

This will a weekly check-in and status update regarding my work on F2PY. The reason being that I have implemented or study something new that I feel worth sharing in detail. I continued working on F2PY's frontend, mostly debugging and improving some facets. Here are the changes:- 

### What did I do this week?

I am so happy to say that I have completed a working and functioning argparse frontend for F2PY. If you had read my last week's post, I had said that I still had to debug and implement a few changes for F2PY.

Specifically:-

1. F2PY's compilation tests were failing, because there were some bug in --f2cmap flag.
2. F2PY would not compile .pyf.src files.
3. I forgot to add version mentioning flag.

I have fixed all the bugs and updated my one and only draft PR: https://github.com/numpy/numpy/pull/21923

All the tests are working correctly and the new f2pyarg should work exactly as f2py2e used to work. Also I found a few bugs that I will report and add fix for. These bugs are:

1. No test for verifying F2PY will work with .pyf.src files.
2. F2PY's test_kind test fails on Max OSX M1 chip.

Also, Rohit and I decided to drop one of our original goals. Before GSoC started we thought it would be a good idea to replace the current class based tests of F2PY to modern fixtures based tests. But we dropped it as the class based structure was performing very well, and the classes were responsible for compiling modules for testing. To get a better understanding of the F2PY's test suites structure and working I recommend you this documentation page: [F2PY's test suite](https://numpy.org/devdocs/f2py/f2py-testing.html)

### What will I do the next week?

I need to add some documentation changes and a release note for the changes have taken place. That's probably all I need to do for the next week, if another bug doesn't pop up.

### Did I get stuck anywhere?

No, this week was comparatively a smooth sailing. I had already built the framework last week and, all I needed to do was to fix the bugs.

That's all my friends.