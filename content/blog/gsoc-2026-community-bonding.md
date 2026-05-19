---
title: "GSoC 2026: Community Bonding Period"
date: 2026-05-19
description: "Starting my Google Summer of Code 2026 journey with NumFOCUS and the DeepForest project."
tags: ["gsoc", "numfocus", "deepforest", "computer-vision"]
showToc: true
---

## Getting Selected

I'm excited to announce that I've been selected for **Google Summer of Code 2026** with **NumFOCUS**, working on the [DeepForest](https://github.com/weecology/DeepForest) project!

My project, *"Recovering Historical Image Data Using Automated OrthoRegistration,"* focuses on building an automated pipeline to align historical forest health survey annotations with modern satellite imagery.

## The Problem

The U.S. Forest Service's Aerial Detection Survey (ADS) program has produced tens of thousands of hand-drawn polygons marking forest damage from aircraft. Due to GPS imprecision and flight geometry, these polygons are spatially offset by 50-500 meters from their true locations.

This means we can't directly use these annotations as training data for modern detection models — the labels don't line up with what's actually visible in satellite imagery.

## My Approach

I'm building a hybrid alignment pipeline that combines:

1. **Geometric transformations** — affine and projective corrections based on known control points
2. **Learned displacement estimation** — a self-supervised deep learning model that predicts per-polygon correction vectors
3. **Energy-function optimization** — combining spectral similarity, edge alignment, and spatial coherence to evaluate registration quality

## What's Next

Over the coming weeks, I'll be:
- Setting up the development environment and understanding the existing DeepForest codebase
- Prototyping the initial alignment pipeline
- Starting work on the learned refinement layer

Stay tuned for updates!

---

*This is part of my GSoC 2026 blog series. Follow along with the [#gsoc](/tags/gsoc/) tag.*
