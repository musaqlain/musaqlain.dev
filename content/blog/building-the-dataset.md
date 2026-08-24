---
title: "There Was No Dataset. So I Built One."
date: 2026-08-22
description: "Building pixel-accurate ground truth for forest damage at 30 cm: why tracing is slow, how 52% of Oregon's damage sitting in one valley broke spatial cross-validation, and why 38% of tiles that were empty by construction were not actually empty."
summary: "The U.S. Forest Service recorded 48,445 damage features across Oregon in 2024, and none of them line up with the trees. Building the dataset to fix that turned out to be the project, not the preparation for it."
tags: ["gsoc", "numfocus", "deepforest", "dataset", "annotation", "remote-sensing", "computer-vision", "forest-health"]
showToc: true
---

<aside class="series-link">
  <span class="series-kicker">Part of a series</span>
  <p>This is the dataset half of my Google Summer of Code 2026 project with NumFOCUS and DeepForest.
  The results and the headline finding are in the
  <a href="/blog/gsoc-2026-final-report/">final report</a>.</p>
</aside>

The U.S. Forest Service Aerial Detection Survey recorded 48,445 damage features across Oregon in
2024. Surveyors sketch them from a moving aircraft: a hand-drawn shape on a tablet, marking roughly
where a patch of dying forest is. They are the best record of forest damage that exists at that
scale.

None of them line up with the trees.

My Google Summer of Code project was to fix that: take a coarse survey polygon and return a boundary
that follows the actual damage. Which requires knowing where the actual damage is. Which requires a
dataset of survey polygons paired with pixel-accurate truth.

That dataset did not exist. Building it turned out to be the project, not the preparation for it.

## Tracing is slow, and the slow part is not the drawing

The first tiles were done by hand in Labelme, one at a time. Fetch 30 cm imagery around an ADS
polygon, load it, draw the damage boundary as I read it from the image.

The drawing is not the bottleneck. The decision is.

A patch of beetle-killed conifer is not a shape with an edge. It is a scatter of dead crowns that
gets denser toward the middle and thins out at the margin, mixed with green trees that are still
alive. There is no line. You have to decide what counts: trace a region containing mostly dead
trees, or trace the dead trees themselves. Trace tightly and you produce a hundred small polygons
per tile and no consistency between tiles. Trace loosely and you label healthy forest as damage.

I settled on damage regions rather than individual trees, which matches what the survey is trying to
record and what a downstream user would want. It also means the labels are deliberately partial. The
annotator traces damage clusters, not every dead tree, so every IoU in this project is a lower bound.
That is stated in the results rather than hidden, because pretending otherwise would make the numbers
look better than they are.

The first batches were small. Around 48 tiles, then roughly double that. Enough to train something,
not enough to trust it.

## Half of Oregon's damage is in one valley

This is the part I would have got wrong without my mentors.

The obvious way to grow a dataset is to annotate more of it. Josh Veitch-Michaelis pushed back on
that after the midterm, and the push was about *which* tiles, not how many. So I measured the pool
I was sampling from.

Roughly **52% of the Moderate-or-denser Mortality polygons in Oregon sit inside a single
central-Cascades Douglas-fir beetle outbreak.** The distribution is not merely uneven. It is one
event with a long tail.

That has a consequence I had already seen without understanding it. The evaluation uses spatial
cross-validation, where folds are split by geography so the model cannot be tested on a tile that
looks identical to one it trained on. With the original sampling, the fold sizes came out as
`[42, 4, 2, 7, 2]`. Three of the five folds held fewer than eight tiles. Those are not folds. They
are rounding errors with error bars.

Sampling "diverse by damage agent", which is what the first version of the script did, does not fix
it. The agents are themselves geographically clustered, so you get variety in the label column and
the same valley in the coordinates.

[`data_prep/build_diverse_seed_tiles.py`](https://github.com/musaqlain/ads-forest-damage-segmentation/blob/master/data_prep/build_diverse_seed_tiles.py)
fixes it directly. It KMeans-clusters the candidate polygon
centroids into 10 spatial clusters, roughly matching the number of real outbreaks in the state, then
**round-robins across those clusters** when selecting tiles: smallest cluster first, easiest tile
first within each. The thin clusters get drained before the dominant one is allowed to contribute
its bulk.

The pool supports it. After filtering to surveyor-drawn polygons of Moderate severity or denser,
1,047 candidate tiles remain, spread across all 10 clusters. There was never a shortage of
geographic diversity. There was a sampling procedure that threw it away.

Severity filtering matters here too, and it is a labelling decision rather than a data-cleaning one.
The severity field records the *density* of dead trees inside a polygon, not its size. Light damage
at 4 to 10% dead is typically a large polygon holding a sparse scatter, which means zooming around
hunting for individual dead crowns and producing slow, noisy annotation. Those were dropped.

## A damage dataset needs verified healthy ground

A model trained only on damage tiles learns what damage looks like without ever learning what it does
not look like. It will find damage everywhere, because it has never been shown ground where the
correct answer is nothing.

Josh asked specifically for confirmed negatives, and the word confirmed is doing real work. A random
empty tile is not evidence of absence. It might be damaged ground nobody flew over.

The sampling geometry encodes that. A negative tile centre must sit **inside the `SURVEYED_AREAS`
footprint**, meaning a surveyor actually flew it and found nothing, with the `NOT_FLOWN` holes
subtracted. It must be at least **150 m from any damage feature of any type**. And accepted negatives
must be at least **800 m apart**, so they sample the state rather than clustering in one meadow.

Negatives are cheap in a way damage tiles are not. They need verification, not tracing. Nobody draws
anything. You look, and you agree or you do not.

## Reviewing them found that 38% were not clean

I swept the first 55 negatives by eye, expecting a formality.

**Roughly 38% were not clean.** Some held real standing dead trees the survey had not recorded.
Others were unusable: blank imagery, water, or ground where I could not tell.

That number changed the design of everything downstream. If more than a third of tiles that are
empty *by construction* are not actually empty, then an unreviewed negative cannot be trusted as an
empty training example. Not as a matter of caution. As a matter of measurement.

So [`annotation/dataset_manifest.py`](https://github.com/musaqlain/ads-forest-damage-segmentation/blob/master/annotation/dataset_manifest.py)
enforces it. An auto-generated negative is **excluded from
training until a human has looked at it**. Not excluded because it is bad. Excluded because it is
unverified. The manifest keeps an explicit `NEG_REVIEWED` set, and a tile outside that set does not
train, no matter how empty its mask is.

## The tooling is what made 95 tiles reviewable in an afternoon

[`annotation/review_tiles.py`](https://github.com/musaqlain/ads-forest-damage-segmentation/blob/master/annotation/review_tiles.py)
runs the sweep. Its design goal was to remove every decision that is not
about the image.

There are three verdicts, and the label you pick in Labelme *is* the verdict:

- **Healthy**: draw nothing, press next. This is the common case and it costs zero effort.
- **Damage**: draw a polygon, label it `damage`. The tile becomes a positive.
- **Reject**: draw a box, label it `reject`. The tile is dropped from training entirely.

Batches are resumable, and tiles are staged **most-open-first**, because open bare ground is where
the model false-alarms. Stopping a sweep halfway still fixes the tiles that matter most. Openness is
measured with excess green, `2G − R − B`, a standard way of separating soil from canopy.

One design detail took a while to get right and is worth stating, because it is the kind of thing
that silently wastes a day of human attention. **A clean sweep has to write a verdict somewhere.**
If reviewing a tile and finding it healthy produces no record, the manifest keeps excluding it as
unverified, and the review changes nothing at all. So confirming a batch appends every tile you swept
to the reviewed set, not just the ones you drew on.

## Annotation infrastructure is software, and it has bugs

Two that cost real time, both found mid-sweep:

A resume bug meant only tiles Labelme had re-saved were marked as done. Since a healthy verdict saves
nothing, a clean batch would have been re-staged forever. I found it because the review kept handing
me tiles I recognised.

And a comment sitting inside a Python list of tile IDs was being parsed as data. A dated note like
`# 2026-07-18 beetle batch` contributed `2026`, `-7` and `-18` as tile IDs, which were then written
back to the file as though I had curated them. Harmless in the end, because no tile has those IDs, but
it corrupted a curation list and I had to verify no real IDs were lost.

Neither is exotic. Both are the ordinary consequence of treating annotation tooling as a script rather
than as software.

## Where it ended

| Measure | Count |
|---|---|
| tiles fetched at 30 cm | 587 |
| used for training | **284** |
| damage sites traced by hand | 151 |
| negatives verified damage-free by eye | 133 |
| crops at a fixed 0.60 m/px | **2,716** |

The training set grew 142 tiles, then 206, then 284, and the run history records every model trained
on each. Every one of those 284 tiles was looked at by a person. The 303 that did not make it were
dropped for a stated reason.

The headline result of this project is that pixel scale, not data volume, was the bottleneck. That
finding is only trustworthy because the folds behind it are geographically real, and the false-alarm
rate behind it is measured against ground somebody actually checked.

Both of those are dataset properties. Neither shows up in the metrics table.

<aside class="series-link">
  <span class="series-kicker">Next in the series</span>
  <p><a href="/blog/bare-ground-and-the-survey/">The Survey Missed What the Model Saw</a> &mdash; what
  those verified-healthy tiles revealed about the model's biggest failure mode, and about the survey
  itself.</p>
</aside>

---

*Follow along with the [#gsoc](/tags/gsoc/) tag.*
