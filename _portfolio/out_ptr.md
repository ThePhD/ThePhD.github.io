---
layout: post
title: out_ptr
feature-img: "assets/img/portfolio/out_ptr.png"
img: "assets/img/portfolio/out_ptr.png"
date: October 10th 2018
tags: [C++, out_ptr, performance, python, Portfolio, 🚌, ⌨️]
---

The `out_ptr` project was started to measure the performance of a common C++ abstraction used from the smallest startups to the biggest million-line codebases all across the world for `std::unique_ptr`. Along the way, the design was modified to work with `std::shared_ptr` and `boost::shared_ptr`, allowing individuals to leverage this simple and abstraction to develop cleaner, safer APIs.

It was eventually rolled into a [C++ Standards proposal](https://wg21.link/p1132)! (See other proposals [here](/portfolio/standard)!)

This project was also the first one where I thought a lot more closely about how to properly represent the data. Learning how to use python to great effectiveness for visualization with matplotlib allowed me an easy way to understand the data being measured, and also taught me a lot about how you present data to others.

Check out the code [here](https://github.com/ThePhD/out_ptr), and check out the `out_ptr` tag for some blog posts!
