---
title: "The Survey Missed What the Model Saw"
date: 2026-08-21
description: "Two failure modes that looked like opposites turned out to be one rule: pale, un-vegetated ground means damage. Diagnosing it, correcting an analysis I had got wrong, and finding evidence that the aerial survey itself under-detects mortality on open ground."
summary: "The model painted damage all over healthy open ground and went silent under closed canopy. Two failures pointing in opposite directions — and one mistake behind both. Including the part where the measurement contradicted my own write-up."
tags: ["gsoc", "numfocus", "deepforest", "computer-vision", "remote-sensing", "model-diagnostics", "forest-health", "machine-learning"]
showToc: true
---

<aside class="series-link">
  <span class="series-kicker">Part of a series</span>
  <p>This is the failure-analysis half of my Google Summer of Code 2026 project with NumFOCUS and
  DeepForest. The results and the headline finding are in the
  <a href="/blog/gsoc-2026-final-report/">final report</a>.</p>
</aside>

The model had two problems, and they looked like opposites.

On open, dry ground it painted damage everywhere, flooding whole images with predictions on forest
that was demonstrably healthy. Under closed canopy it did the reverse: it said nothing at all. About
14% of the images that did contain dead trees, 268 of 1,926, came back completely empty. An
empty prediction scores zero recall (recall = the share of real damage the model actually found), so
those images were a straight loss.

Two failures, pointing in opposite directions. It took a while to see they were the same mistake.

## What the model actually learned

Some background on the task. The U.S. Forest Service runs an **Aerial Detection Survey** (ADS): each
year, surveyors fly over the forest and sketch polygons around damage they can see from the aircraft.
The polygons are roughly right about *where* damage is and wrong about its *shape*. This project trains
a segmentation model to redraw those boundaries from 30 cm aerial imagery.

Here is the difficulty. From directly overhead, a dead conifer and bare soil are nearly the same colour.
Both are pale. Both are not-green. What separates them is texture and crown shape, which is fine detail.

The model resolved that ambiguity in the cheapest way available to it: **pale, un-vegetated ground means
damage.**

That single rule produces both failures. Where the ground really is bare, it fires when it should
not. Where the canopy is closed and dark, it stays quiet even when dead trees are present, because the
cue it learned isn't there.

## Testing the idea instead of asserting it

A story that explains everything is a warning sign, not a result. So the rule got measured.

Every image was scored with an **excess-green index**, `2G − R − B`, a standard way to separate soil
from canopy using only the visible bands. Call the resulting number *openness*: high means mostly bare
ground, low means mostly vegetation. Then performance was compared across that scale.

The pattern was monotonic. In the most open quarter of images, recall climbed to 0.61 and the share of
images where the model gave up entirely fell to 6.7%. In the most closed quarter, recall was 0.41 and
20–23% of images came back silent. The model fires *more* on open ground and *less* under canopy. Same
rule, two symptoms.

## Where the analysis was wrong

An earlier version of this write-up claimed both failure modes happened in open woodland. That claim
came from scrolling through contact sheets of the worst predictions and noticing that a lot of them
looked like dry, open country.

Measurement contradicted half of it.

The silent images are in **closed** canopy, not open. Mean openness for silent images was 0.45 against
0.52 for firing ones in one run, and 0.44 against 0.53 in the next. Both p < 0.0001, replicated. And
openness barely predicts segmentation quality at all: rho = −0.013 (p = 0.56) in one run and +0.063
(p = 0.006) in the next. Noise either side of zero.

Three dozen thumbnails looked conclusive and were not. The claim got corrected rather than defended,
and the correction is still in the repository, because the failure mode is worth more than the tidy
version.

## The decisive test

Most of these measurements have an escape hatch: the labels are partial. Annotators traced damage
clusters, not every individual tree, so a "false alarm" might be real damage nobody drew.

One measurement has no such excuse. The images independently verified to contain **no damage at all**.
These are sampled inside surveyed areas, at least 150 m from any mapped damage, and then checked by eye.
If the model paints pixels there, it is wrong. Full stop.

On 412 verified-healthy images:

> Open images had **several times more** of their pixels falsely painted than closed-canopy ones.
> Two runs: 15.5% vs 1.2%, and 13.2% vs 5.1% (rho = +0.46 and +0.34, both p < 0.0001).

The direction replicates. The exact ratio does not. The open figure is stable (15.5%, 13.2%) while the
closed-canopy baseline moves, which is what swings the ratio from 13x to 2.6x. So the honest phrasing is
"several times worse", never a fixed multiple.

## Then look at the training data

If the model believes bare ground means damage, the obvious next question is what it was shown.

Scoring every *training* image on the same openness scale answered it immediately:

> Among training images more than half bare ground: **61 damage against 12 healthy. 5.1 to 1.**

The model had almost never seen pale, open ground that was *not* damage. Given that diet, "bare ground
means damage" isn't a flaw in the model. It's an accurate summary of what it was taught. The model was
right about its training set and wrong about Oregon.

## The fix needed no new imagery

Ninety-five verified-negative images had already been downloaded months earlier and were sitting unused
on disk. They were excluded from training for exactly one reason: no human had looked at them yet, so
they could not be trusted as empty.

Reviewing them took a few hours in Labelme. It moved the negative pool from 60 to 133 images and the
open-ground balance from 5.1 : 1 to **1.68 : 1**.

Three fresh runs, against three baseline runs:

| Measure | before | after |
|---|---|---|
| open-ground damage : healthy | 5.1 : 1 | **1.68 : 1** |
| false alarms on healthy ground (commission) | 8.8% | **6.6% ± 1.0%** |
| recall | 0.50 | **0.52 ± 0.02** |
| region IoU | 0.290 ± 0.013 | 0.268 ± 0.004 |

*Commission* is the share of pixels the model paints as damaged on ground verified to have no damage.
*IoU* (intersection over union) measures how closely the predicted region matches the traced one.

Every one of the three new runs has lower commission than either baseline run: 5.48, 6.65, 7.57 against
8.36 and 9.15. Every one also has lower IoU: the best new run, 0.2721, still sits below the worst
baseline run, 0.2748. Complete separation in both directions.

That reads well until you count the runs. Three against three means a permutation test cannot get below
p ≈ 0.05–0.10 no matter how clean the separation looks. **This is a reproducible trade, not a proven
one.** It bought a 2.2-point drop in commission for 0.022 IoU, and recall did not pay for it.

## Three things that temper it

**A 6% commission target was set before the experiment, and the mean missed it.** The result was 6.6%.
One run of three cleared the bar at 5.48%; the other two did not, at 6.65% and 7.57%. This gets reported
as a near-miss, not a pass. Deciding the threshold afterwards would have made the number meaningless.

**The bias is reduced, not removed.** Open images still get more of their pixels falsely painted than
closed ones: 8.3% vs 2.7%, 9.1% vs 4.1%, 9.9% vs 5.2% across the three runs, ratios of 3.1x, 2.2x and
1.9x, with rho = +0.283, +0.306, +0.295, all p < 0.0001. The direction replicated every time. More
negative examples is no longer the lever either; at 1.68 : 1, adding more would start costing recall.

**The comparison flatters the baseline, not the fix.** The verified-healthy test set nearly doubled, from
412 images to 790, *and* was deliberately loaded with the open ground the model finds hardest. So 6.6%
was earned on a harder exam than the 8.8% it replaced. For scale, the ADS polygon itself paints 13.5% of
pixels on those same 790 verified-healthy images. The model is less than half as wrong as the survey it
is correcting.

## The reviewing turned up something better than the metric

Here is the part that did not come from a number.

Going through those open-ground images one at a time, roughly **half of the ones the survey had recorded
as undamaged contained visible standing dead trees.** Grey-stage mortality: trees dead long enough to
have dropped their needles, leaving pale, see-through crowns of bare branches that cast tree-shaped
shadows. Once you have seen one you cannot unsee them. They were reclassified rather than trained as
healthy.

That points the opposite way from everything above.

If the survey systematically under-detects mortality on open ground, then some share of what this project
has been calling model false alarms may be the model finding real damage and being scored against an
incomplete label. The metric would call that a failure. The imagery suggests otherwise.

This is a hypothesis, not a result. Confirming it needs field data, not more imagery. Somebody has to
walk into those stands and count. But of everything produced here, it is the most useful thing to hand
back to the survey team, and it is the one I would most like to be shown wrong about.

The model learned that bare ground means damage because that is what we taught it. The interesting
possibility is that on open ground, it may have been learning something we did not know we were
teaching.

<aside class="series-link">
  <span class="series-kicker">Also in the series</span>
  <p><a href="/blog/building-the-dataset/">There Was No Dataset. So I Built One.</a> &mdash; how the
  verified-healthy tiles behind this analysis were sampled, reviewed, and kept out of training until a
  person had looked at them.</p>
</aside>

---

*Follow along with the [#gsoc](/tags/gsoc/) tag.*
