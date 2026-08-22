---
title: Job Hunting and ChatGPT - A Developer's Journey 
published: false
description: Preface to give you a context of how I ended up making a Chrome extension for scraping
tags: 'Scrapping'
cover_image: ./assets/extension_chapter_1_img2.png
---

"A phrase that has stayed with me to this day is that job hunting as a developer is a numbers game."

Warning: This article is not about technical guides, but rather a preface to give you a context of how I ended up making a Chrome extension for scraping.

### The Tech Sector is... hard to explain
![extension_chapter_1_img1.png](./assets/extension_chapter_1_img1.png)

Unlike many, I had an easy start in my programming career. My first and second jobs almost fell into my lap. But from my third job onwards, it was like hitting a concrete wall.

When my second job ended, the pandemic began three weeks later, leading to a hiring freeze. After 1, 2, 3, 4 months, things didn't improve. I started job hunting again and it took 8 months. I wondered if I was the problem.

Looking back, I lacked certain skills, but the real issue was the broken recruitment system. At 15, I was already soldering electronic boards and programming chips in C++. Yet at 21, with more experience, I struggled to find a job. It was frustrating and I don't think it will be the last time.

### Scrapping and Form Filling
Job hunting was tough. I had responsibilities: no children, but three cats, a beautiful dog, and a wife. What would you do if you lost your job, had a family to feed, and knew the market was laying off?

I practiced LeetCode problems, but that's useless without technical interviews. To have stressful interviews about data structures you won't use daily, you first need an invitation to the meeting.

I'm a developer; I create solutions. Manual job applications are time-consuming.

![extension_chapter_1_img2.png](./assets/extension_chapter_1_img2.png)

The process involves:

Uploading a CV and then filling in the same information manually.
Sifting through job feeds to find relevant ones.
Communicating with recruiters, refilling forms manually.
Finding a job that matches your profile but... it's in India. Why is it listed in Spanish?
So, I started creating solutions.

Last year, I automated my job search process. If job hunting is a numbers game, the more applications and interviews, the better. Ironically, I didn't create any useful tools - at least none that were easy and adaptable.

My First Use Case: LinkedIn's limited filters were a problem. Many job posts were for or from India, which didn't interest me. I wanted an automatic solution.

![extension_chapter_1_img4.png](./assets/extension_chapter_1_img4.png)

I looked for a scraper to save information and filter it automatically with ChatGPT. Not finding what I wanted, I created my own solution.

Puppeter as a Mediocre Scraper
I had done small scraping projects before and chose to use Node instead of an existing tool. Scraping for cars on a website without login requirements is different from scraping LinkedIn, where you need to be logged in and avoid getting banned.

The Final Question: How to scrape LinkedIn? Most tools use Chromium with Puppeter, but logging in, especially with Google, and possibly dealing with CAPTCHAs, was overly complex. The easiest way was to control the browser directly since I was already logged in.

But, when I had to change PCs and redo the setup for Puppeter, I realized it was time to move on to something else.

Chrome Extensions: A Hassle
![extension_chapter_1_img3.png](./assets/extension_chapter_1_img3.png)

Chrome extensions consist of three parts:

The Popup
The Backend
The Content
The popup and backend are fully controllable, allowing installations, WebSocket connections, and requests. The content is JavaScript you can run in your browser tab, but with limitations.

My Conclusion: I'll discuss the best way to work with Chrome extensions and ChatGPT in the next article. The less code the extension has, the better. Gather data through the content, communicate it to the backend, then connect with sockets to a Node server, linking the popup to the socket and server.