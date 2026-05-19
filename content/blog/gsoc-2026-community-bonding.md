---
title: "GSoC 2026: My Journey to NumFOCUS & DeepForest"
date: 2026-05-19
description: "How I got selected for Google Summer of Code 2026 with NumFOCUS, my community mentorship journey, the project I'll be working on, and what's ahead."
tags: ["gsoc", "numfocus", "deepforest", "computer-vision", "remote-sensing", "open-source"]
showToc: true
---

## About Me

Hello! My name is Muhammad Saqlain. I'm a Master's student at the University of Education, Lahore, Pakistan, and I'm excited to share that I've been selected for **Google Summer of Code 2026** with [NumFOCUS](https://www.numfocus.org/), working on a project called [DeepForest](https://github.com/weecology/DeepForest). I will be working under the mentorship of [University of Florida](https://www.ufl.edu/) researchers this summer 🎉🥳

This is actually my second time being selected for GSoC. In 2025, I was a contributor with [Google Chromium](https://summerofcode.withgoogle.com/archive/2025/projects/LzP98Ubl). But this time, since I'm a Master's student, I specifically wanted a research-based project. The one I chose is purely research-oriented. It's not the kind of problem an AI agent can just solve for you, because the problem itself is unique and requires genuine research to make progress. However, as an AI lover, I personally use AI in my daily tasks and am also interested in working with AI. Despite that, I chose this project because it aligns with my long-term research goals.

I submitted my proposal to [NumFOCUS](https://www.numfocus.org/), which is one of the pioneering organizations in research, data, and scientific computing. This year, 23,371 proposals were submitted from 131 countries (a selection rate of 4.88%, and NumFOCUS has an even lower selection rate). To put this into perspective, NumFOCUS is the backbone of the global data science ecosystem, sustaining tools like [NumPy](https://www.numpy.org/) and [Pandas](https://www.pandas.dev/), and is heavily backed by tech giants like [Google](https://www.linkedin.com/company/google/) and [Meta](https://www.linkedin.com/company/meta/). I will be specifically working with the University of Florida [Weecology lab](https://www.weecology.org/) on DeepForest, a pioneering project using state-of-the-art computer vision to analyze high-resolution ecological and remote sensing imagery.

I remember my mentors interviewing me just 36 hours before the official GSoC contributor selection date because NumFOCUS is brutally competitive. I had 4 interviews with the NumFOCUS team totaling 6+ hours, where we discussed everything from PRs and prototypes to debugging, learning new computer vision frameworks in a very short time, and presenting our findings. All of these things were evaluated in a points-based approach by NumFOCUS senior administrators to make the final decision. But Alhamdulillah, the hard work paid off, and I successfully secured my place to work on this incredible research!
My mentors for this journey will be [Ethan White](https://www.linkedin.com/in/ethanwhite/), [Henry Senyondo](https://www.linkedin.com/in/henrysenyondo/), [Josh Veitch-Michaelis](https://www.linkedin.com/in/jveitchm/), and [Ben Weinstein](https://www.linkedin.com/in/ben-weinstein-9423034/).

I am very thankful to [Zeeshan Adil](https://www.linkedin.com/in/zeeshanadilbutt/), who motivated me to find a GSoC project that is research-oriented, which is how I found NumFOCUS in the first place.

## My Community & Mentorship

Apart from my own GSoC work, I run a community of approximately 600 students focused on spreading awareness about open source programs among Pakistani youth through the [Dev Weekends](https://www.devweekends.com) platform. In Pakistan, Dev Weekends is the only non-profit organization that provides this kind of mentorship platform for GSoC.

In 2025, while I was preparing for my own GSoC selection with Chromium (Alhamdulillah, I got selected), two more students were selected under my guidance:

| Student | Organization | Project |
| --- | --- | --- |
| [Muhammad Salman](https://www.linkedin.com/in/msalman199/) | [Fossology](fossology.org) | [Official GSoC Project](https://summerofcode.withgoogle.com/archive/2025/projects/MjOyiOj7) |
| [Yash Israni](https://www.linkedin.com/in/yashisrani/) | [Chinese OSPP (Open Source Promotion Plan)](https://blog-en.summer-ospp.ac.cn/) | [Chinese OSPP (Open Source Promotion Plan)](https://www.linkedin.com/posts/yashisrani_this-summer-i-was-selected-for-the-open-activity-7372530838594453504-d5g8/) |

This year, that number grew to **11 selections**, and I am so proud of every single one of them. I have been mentoring these students since November-December along with [Muhammad Salman](https://www.linkedin.com/in/msalman199/). And [Zeeshan Adil](https://www.linkedin.com/in/zeeshanadilbutt/) bhai helped each one of us in the backend. Honestly, without him, most of us would not have been selected. He gave each student personal attention, kept us motivated, and proposed an organization to each one of us:
<!-- represent it in table -->
| Student | Organization | Project |
| --- | --- | --- |
| [Kamran Haq](https://www.linkedin.com/in/kamran-ul-haq/) | [FLARE](https://www.linkedin.com/company/flareai/) | [Official GSoC Project](https://summerofcode.withgoogle.com/programs/2026/projects/MNBryJ8N) |
| [M Junaid Shaukat](https://www.linkedin.com/in/junaiddshaukat/) | [Apache Software Foundation](https://www.linkedin.com/company/apache-software-foundation/) | [Official GSoC Project](https://summerofcode.withgoogle.com/programs/2026/projects/x13a0jGL) |
| [Muhammad Aqib](https://www.linkedin.com/in/m-aqib-nawab/) | [C2SI](https://www.linkedin.com/company/c2siorg/) | [Official GSoC Project](https://summerofcode.withgoogle.com/programs/2026/projects/7sf7V0bk) |
| [Talha Asif](https://www.linkedin.com/in/talha-asif7/) | [Drupal](https://www.linkedin.com/company/drupal/) | [Official GSoC Project](https://summerofcode.withgoogle.com/programs/2026/projects/C09HAZpF) |
| [Mateeb Haider](https://www.linkedin.com/in/mateeb-haider/) | [FOSSASIA](https://www.linkedin.com/company/fossasia/) | [Official GSoC Project](https://summerofcode.withgoogle.com/programs/2026/projects/eyJnXkkA) |
| [Muhammad Rebaal](https://www.linkedin.com/in/muhammad-rebaal-34936616a/) | [ESoC](https://www.linkedin.com/company/esoc-summer-of-code/) | [Official GSoC Project](https://summerofcode.withgoogle.com/programs/2026/projects/JtU8xS6c) |
| [Muhammad Saqlain](https://www.linkedin.com/in/musaqlain/) | [NumFOCUS](https://www.linkedin.com/company/numfocus/) | [Official GSoC Project](https://summerofcode.withgoogle.com/programs/2026/projects/jLoU4DCV) |
| [Abdullah Waleed Ahmed](https://www.linkedin.com/in/abdullah-waleed-ahmed-774890343/) | [BRL-CAD](https://www.linkedin.com/company/brl-cad/) | [Official GSoC Project](https://summerofcode.withgoogle.com/programs/2026/projects/QdVvPjPX) |
| [Muneeb Ahmad](https://www.linkedin.com/in/muneeb-ahmad-se/) | [Invesalius](https://www.linkedin.com/company/invesalius/) | [Official GSoC Project](https://summerofcode.withgoogle.com/programs/2026/projects/ipVIM23p) |
| [MuhammadAli Arif](https://www.linkedin.com/in/aliarif-se28/) | [EMBL-EBI](https://www.linkedin.com/company/embl-ebi/) | [Official GSoC Project](https://summerofcode.withgoogle.com/programs/2026/projects/brBab9aI) |
| [Yash Israni](https://www.linkedin.com/in/yashisrani/) | [LFx Mentor](https://www.linkedin.com/company/lfx-linux-foundation/) | [Official LFX Project](https://mentorship.lfx.linuxfoundation.org/project/c5b8d1fa-b75a-4f88-a63b-e6487dc7e39b) |

## The Project

The U.S. Forest Service runs one of the longest-running airborne forest health monitoring programs in the world called the **Aerial Detection Survey (ADS)**. Every year, trained observers fly over forests and hand-draw polygons on maps marking areas of tree damage like bark beetle outbreaks and drought mortality. Over the decades, this has produced tens of thousands of annotated polygons across the U.S.

The problem is that these polygons are **spatially misaligned**. They're off by 50 to 500 meters from where the actual damage is, because the observers are drawing from a moving aircraft with imprecise GPS. On top of that, there's often a 1 to 3 year gap between when the survey was done and when the satellite imagery (NAIP) was captured. So you can't just use these polygons as training data for deep learning models because the labels simply don't line up with what's visible in the imagery.

My project aims to **automatically correct these spatial offsets** so that the ADS archive can finally be used as training data for forest health detection models. You can find the full project description [here](https://github.com/weecology/retriever/wiki/GSoC-2026-Project-Ideas#proposal-2-recovering-historical-image-data-using-automated-ortho-registration-and-image-matching). This is a high-impact and largely unsolved problem.

## My Approach

The core design philosophy I adopted is: *"Use the structure of the problem to constrain the search space, rather than hoping a CNN will discover it from raw pixels."*

The pipeline has multiple stages:

1. **Seed Data Construction:** I started by manually verifying 11 polygon-NAIP pairs in QGIS, carefully matching each ADS survey year to the correct year-specific NAIP server. This temporal matching turned out to be critical because using the wrong year silently corrupts everything.

2. **ProximityAlign Hybrid Energy Alignment (Classical, no training needed):** I adapted a contour-to-boundary energy function from a paper on urban vector map alignment. Pure contour matching failed because damage boundaries are everywhere in forest tiles, so I built a hybrid energy function combining 70% inside-outside contrast, 20% contour-to-boundary distance, and 10% out-of-bounds penalty. This improved 4 out of 11 samples, with the mean IoU going from 0.082 to 0.231.

3. **Learned Refinement (GSoC research phase):** The classical approach handles cases with a clear spectral signal. For the remaining cases where the spectral difference is within noise, a learned model is needed. The architecture will be decided with mentor guidance, with options including self-supervised displacement estimation, residual translation prediction, or learned feature maps.

4. **Scaling & DeepForest Fine-Tuning:** Once the alignment pipeline is validated, apply it to all ~48,000 Oregon ADS polygons, export as training data, and fine-tune DeepForest for forest health detection on NAIP imagery.

## Implementation Plan

The GSoC 2026 coding period runs from **May 27 to August 25** (12 weeks):

| Period | Focus |
|--------|-------|
| **Community Bonding** | Sync with mentors on Stage 2 architecture, review Stage 1 results, expand seed dataset |
| **Weeks 1–2** | Optimize and scale Stage 1 (target ≤5s/sample), run on 500+ polygons to establish baseline |
| **Weeks 3–4** | Build Stage 2 learned refinement prototype, initial training runs |
| **Weeks 5–6** | Iterate on Stage 2, combined evaluation (midterm checkpoint) |
| **Weeks 7–8** | Production scaling to all ~48K Oregon ADS polygons, confidence filtering |
| **Weeks 9–10** | DeepForest fine-tuning on aligned dataset, evaluation against base model |
| **Week 11** | Full evaluation, diagnostic visualizations, document failure modes |
| **Week 12** | Documentation, blog post, reproducibility guide, submit deliverables |

---

Thank you for reading! I'll be posting updates every two weeks throughout the GSoC period. Stay tuned!

*Follow along with the [#gsoc](/tags/gsoc/) tag.*
