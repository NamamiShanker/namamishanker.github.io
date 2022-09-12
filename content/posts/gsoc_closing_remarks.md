+++ 
draft = true
date = 2022-09-12T19:22:20+05:30
title = "GSoC'22 closing remarks and Work Report"
description = ""
slug = "gsoc-closing-remarks"
authors = ["Namami Shanker"]
tags = ["GSoC'22", "F2PY"]
categories = []
externalLink = ""
series = ["GSoC'22"]
+++

# GSoC'22 Work Submission and closing remarks

## Closing remarks

Participating in Google Summer of Code 2022 with NumPy was more than just contributing to a summer coding internship for me. I remember walking around my college hostel along with my friend during the first year of my engineering in 2019 discussing what do we want to achieve over the course of our undergraduate education.  Back then too, I remember that my first wish ever, even before I knew what open sources was to participate in the Google Summer of Code.

We study in a relatively new institution and our seniors were just graduating and the alumni were beginning to leave their mark in the world. No one had ever participated in a GSoC from our college, and barely anyone knew what it was. Nevertheless, with the help of YouTube and GitHub I got exposure to Open Source, with the GirlScript Summer of Code being the first Open Source program I participated in. Then I started participating in SciPy's pull requests, fixing its bugs and improving documentation in 2021.

Slowly and steadily, I began to talk to maintainers and attend weekly Community Call meetings in the second half of the year 2021. I had never encountered such lovely and welcoming communities ever before. Neither at gaming discord servers nor at school, college or anywhere. They promoted curiosity and rewarded questions and contributions. It is as perfect a learning environment and support group a person can imagine.

From the start of my exposure, I loved open source. I loved talking to these people building their exciting projects, learning from them, being a part of their communities and contributing to these projects at the cutting edge of software development.

I remained an active contributor to SciPy till December 2021. In January 2022, SciPy posted their [idea list for GSoC'22](https://github.com/scipy/scipy/wiki/GSoC-2022-project-ideas). Being an electrical engineering student, I first tried to go for [this signal processing project](https://github.com/scipy/scipy/wiki/GSoC-2022-project-ideas#object-oriented-design-of-digital-filtering-in-scipysignal) but within a first couple of days discussing this project with the maintainers I could tell that there was no mentor for this project. So I switched my project preference to [Restructuring F2PY frontend](https://github.com/scipy/scipy/wiki/GSoC-2022-project-ideas#restructuring-the-f2py-frontend) which I knew nothing about.

Now, I know a lot of people would disagree with my approach towards GSoC. GSoC is viewed as a program where you get a chance to improve projects you are interested in and have knowledge about and get financial renumeration as a stipend for it. Coming clean about the fact that I switched to a different project I had never heard about would seem strange to a lot of contributors and past GSoC participants. 

I appreciate this current notion of GSoC. In fact, I prefer this current idea of GSoC. It ensures that students aren't motivated by the stipend, but by their genuine interest in the project. However, shying away from GSoC just because your favourite organisation didn't participate with the project your wanted to work on is an ill conceived plan. When you are studying for a bachelors degree and have barely any exposure to real world industry or software development experience, you should always grab any oppurtunity, especially something aimed for newcomers as per the new (2022) GSoC guidelines.

Now, I knew absolutely nothing about the project I wanted to contribute to - [the F2PY tool](https://numpy.org/doc/stable/f2py/). I started by contacting the NumPy mentor for the project - [Rohit Goswami](https://rgoswami.me) and asking some questions about how to get started with F2PY, which codebases I should familiarise myself with, what are the project goals etc. One thing that has left an indelible mark on me and probably I will never forget is how welcoming he was towards newcomers asking about `f2py`. I remember he immediately scheduled a call with me, and patiently explained to me all the my initial inqueries and was kind enough to make the meeting notes and share it with me.

I loved my GSoC timeline and contributions, but more than that, I am happy about having met my mentor and having the oppurtunity to work with him. I learned a lot of things from the way he believes in pragmatically approaching problems to puzzle out solutions. At the same time, he has an easy vibe around him which makes talking to him and discussing problems much easier. More than just being a GSoC student, he motivated me to be a better student (he having been being an excellent student himself), to think big, work smart and hard, be a kid at heart, and take it easy. My experience throughout GSoC working with my mentor has made me a better individual altogether.

# Work report

You can have a look at my GSoC proposal here  - [Restructuring F2PY frontend and integrationg Meson Build System](https://drive.google.com/file/d/15DIU6pWisuzw8WDOh5cmY9NxzBTUjbNj/view?usp=sharing). My entire work is divided into 2 giant Pull Requests - 

1. https://github.com/numpy/numpy/pull/21923
    - Contains changes to the frontend command line parser.
3. https://github.com/numpy/numpy/pull/22225
    - Consists Meson build system integration code.

These pull requests together involve over 1600 lines of code changes spread accross 132 commits. You can dive through [my blog posts](https://blogs.python-gsoc.org/en/namamishankers-blog/) if you are interested in getting an overview of my changes. I have tried to keep my blog-posts beginner friendly without getting too deep into the nitty-gritty implementation details.

The frontend parser successfully compiles SciPy which was a major goal for us to address. The Meson build system is working, but without many tests beyond documentation examples or benchmarking suites we cannot yet assert that it is feature complete and bug free.

Post GSoC, I plan to work on:-
1. Creating a testing and benchmarking suite for F2PY.
2. Developing F2PY APIs.
3. Implementing derived types support.
4. Start contibuting again to SciPy's stats module.

My contributions and work have just started, and I am grateful to have been welcomed by the larger `f2py` community beyond Rohit, including Dr. Pearu Peterson and Dr. Melissa Weber Mendonça. The NumPy community has also been very accomodating, including Inessa Pawson and Dr. Ralf Gommers (also our sub-org admin).