---
title: "Recovering Forest Damage Boundaries from Aerial Imagery"
date: 2026-08-23
description: "My Google Summer of Code 2026 final report with NumFOCUS and DeepForest: training a segmentation model to redraw U.S. Forest Service aerial survey polygons from 30 cm imagery, and the finding that pixel scale — not data volume — was the bottleneck."
summary: "GSoC 2026 final report. Turning rough, hand-drawn forest-damage polygons into accurate boundaries at 30 cm — including the negative result that ended the original approach, and the preprocessing decision that had been capping the model from the start."
tags: ["gsoc", "numfocus", "deepforest", "computer-vision", "remote-sensing", "segmentation", "forest-health", "machine-learning"]
showToc: true
---

<div class="post-dek">
  <p class="dek-title">Google Summer of Code 2026 &mdash; final report</p>
  <dl>
    <dt>Organisation</dt>
    <dd>NumFOCUS &middot; <a href="https://github.com/weecology/DeepForest">DeepForest</a> / Weecology Lab, University of Florida</dd>
    <dt>Mentors</dt>
    <dd>Ben Weinstein, Josh Veitch-Michaelis</dd>
    <dt>Code</dt>
    <dd><a href="https://github.com/musaqlain/ads-forest-damage-segmentation">ads-forest-damage-segmentation</a></dd>
  </dl>
</div>

## The problem

Every year the U.S. Forest Service flies its Aerial Detection Survey over the western states. A
surveyor looks out of a moving aircraft and draws polygons around forest damage on a tablet. That is
the main record of where insects and disease are killing trees. Oregon alone contributes about 48,000
of those polygons.

They are roughly right about where the damage is. They are wrong about its shape. They are drawn at
speed, from altitude, without a pixel grid. If you want to measure how much forest actually died, you
cannot use them as they are.

There is also 30 cm aerial imagery of the same ground, and it is public.

So the question was: can the imagery turn a rough polygon into an accurate damage boundary?

## The first result was a negative one

The project started on an assumption that turned out to be false. Testing it early was the best thing
that happened in the first month.

The assumption was that the polygons are shifted. Drawn correctly, placed wrong, because the aircraft
did not know exactly where it was. If that were true the fix would be cheap: find the translation,
rotation and scale that moves each polygon back onto its damage, then apply it.

It is not true.

> **The corrections are reshapes, not shifts.** No affine transform recovers the true boundary. The
> polygon's shape is wrong, not just its position.

This is easy to get wrong. If you score the error in the transform's raw parameters, the approach
looks like it is working. Score corner displacement instead, and decompose the recovered rotation and
scale, and you can see that it is not.

That ended the alignment approach and moved the project to segmentation. Stop trying to move the
polygon. Predict the damage region straight from the imagery, and use the polygon only as a weak hint
about where to look.

## Building a dataset that did not exist

There was no pixel-accurate ground truth for this. No public dataset pairs ADS polygons with
hand-traced damage boundaries at 30 cm.

So the dataset became the project:

- Traced damage regions by hand in Labelme. About 48 tiles first, then roughly double that.
- Found that the sampling was geographically lopsided. Around **52% of Oregon's moderate-or-denser
  damage candidates sit in one central-Cascades Douglas-fir beetle outbreak**. Sampling for variety in
  damage agent does not escape it, because the agents cluster geographically too. Fixed by
  KMeans-clustering candidate centroids into 10 spatial clusters and sampling across them
  ([`data_prep/build_diverse_seed_tiles.py`](https://github.com/musaqlain/ads-forest-damage-segmentation/blob/master/data_prep/build_diverse_seed_tiles.py)).
  This came from Josh after the midterm.
- Added confirmed negatives, meaning tiles verified to hold no damage. They sit inside surveyed areas,
  at least 150 m from any damage feature, and at least 800 m from every other tile.
- Built review tooling ([`annotation/review_tiles.py`](https://github.com/musaqlain/ads-forest-damage-segmentation/blob/master/annotation/review_tiles.py),
  [`annotation/dataset_manifest.py`](https://github.com/musaqlain/ads-forest-damage-segmentation/blob/master/annotation/dataset_manifest.py))
  so verifying negatives was a repeatable batch job instead of a slog.

**Final dataset: 587 tiles fetched, 284 used for training. 151 damage sites traced by hand and 133
negatives verified by eye, giving 2,716 crops at a fixed 0.60 m/px.**

One number explains why the review work was worth it. Of the first 55 negatives I checked, about
**38% were not actually clean**. Some held real dead trees the survey had not recorded. Others were
unusable. So an unreviewed negative cannot be trusted as empty, and the pipeline keeps it out of
training until a person has looked at it.

<aside class="series-link">
  <span class="series-kicker">Deep dive</span>
  <p>The full story of how this dataset was built &mdash; the tracing decisions, the geographic sampling
  bug, and why 38% of &ldquo;empty&rdquo; tiles were not empty &mdash; is its own post:
  <a href="/blog/building-the-dataset/">There Was No Dataset. So I Built One.</a></p>
</aside>

## The main finding: pixel scale was the bottleneck, not data volume

If I could keep one result from this summer, it would be this one.

The seed tiles are not the same size on the ground. The smallest covers 180 m across. The largest
covers 1500 m. Every one of them was being resized to the same 384 x 384 image before training. So one
pixel meant anywhere from 0.61 m to 3.68 m of real ground, a 6x spread, while the pretrained model's
features were learned at a fixed 0.60 m/px.

That matters because of what a dead tree looks like from above. A dead conifer and bare soil are
almost the same colour. What separates them is texture and crown shape. A crown is about 9.5 m across,
which is **16 pixels wide at 0.60 m/px**. On a squashed large tile it is **2.5 pixels**, and the
texture is gone. The model was being asked to separate two things that had been made to look
identical.

The fix was to stop resizing. Cut each tile into 384 x 384 pieces at a fixed 0.60 m/px, and take as
many pieces as the tile gives you.

The cleanest evidence needs no statistics:

> **Zero-shot IoU went from 0.040 to 0.108.** Same pretrained weights, no training, same ground. Only
> the pixel scale changed.

Fine-tuned, the same change moved region IoU from **0.116 to 0.257**. Every crop run beat every
whole-tile run, which puts a permutation test on its floor at p = 0.008.

I had assumed the bottleneck was the number of labelled tiles. It was not. It was a preprocessing
decision that had been sitting there since the start.

## Where this stood at the midterm

At the midterm evaluation on 6 July I had all five of the tasks Ben set covered, 57 usable tiles
(47 damage and 10 clean negatives), and a headline region IoU of 0.30.

That 0.30 needs an asterisk, and I put one on it at the time. It came from random cross-validation,
which lets the model be tested on a tile that looks almost identical to one it trained on. The
spatially-blocked number, which is the honest one, was 0.215. My own slide said "truth in between".

So the like-for-like progress is this:

| Measure | midterm (6 Jul) | final (22 Aug) |
|---|---|---|
| tiles used for training | 57 | **284** |
| damage / negative split | 47 / 10 | **151 / 133** |
| region IoU, spatially-blocked | 0.215 | **0.268 ± 0.004** |
| smallest cross-validation fold | 2 tiles | **289 crops** |
| logged runs | not tracked | **29** |

I want to be clear that 0.30 to 0.268 is not a regression. The 0.30 was measured a way I stopped
using because it flatters the model.

The midterm deck also listed four next steps. Three months later, here is what actually happened to
each of them.

| What I said I would do | What happened |
|---|---|
| More labelled tiles, plus copy-paste augmentation for the rare damage class | Got the tiles, 57 to 284. Never used the augmentation, because volume turned out not to be the bottleneck |
| Self-supervised pretraining on the thousands of unlabelled tiles | Dropped. Once the resolution problem was found there was a cheaper thing to fix first, and I would rather report one clean result than start a second track |
| Report spatially-blocked CV as the headline | Done. It is now the only number reported anywhere in the repository |
| Scale from tiles to km-scale polygons via gridded patch-tiling | This became the fixed-resolution crop pipeline, and it turned into the main result of the summer |

The last row is the one I did not see coming. I wrote it down as an engineering step for handling
large polygons. It turned out to be the thing that was holding the whole model back.

## Results

| Setting | region IoU | recall | commission |
|---|---|---|---|
| ADS polygon as-is, no model | 0.115 | | |
| Pretrained weights, zero-shot, wrong resolution | 0.040 | | |
| Pretrained weights, zero-shot, correct resolution | 0.108 | | |
| Fine-tuned, whole tiles resized to 384 px | 0.116 ± 0.027 | 0.33 | ~2% |
| Fine-tuned, fixed 0.60 m/px crops, 30-epoch budget | 0.257 ± 0.020 | 0.47 ± 0.04 | 7.0% ± 2.5% |
| Same, 60-epoch budget | 0.290 ± 0.013 | 0.50 | 8.8% |
| **Same, plus 73 more verified-healthy tiles** (deployed) | **0.268 ± 0.004** | **0.52 ± 0.02** | **6.6% ± 1.0%** |

Every row is a mean over repeated identical runs. The spread is run-to-run training noise, not a
confidence interval. All 29 runs are in `run_history.csv`, including the ones that did not support the
conclusion.

*IoU* is how closely the predicted region matches the traced one. *Recall* is the share of real damage
the model found. *Commission* is the share of pixels it paints as damaged on ground verified to have
none. Tile-level detection, meaning does this crop contain damage at all, reaches ROC-AUC
0.773 ± 0.018.

**On why the deployed model is not the highest-IoU model.** The last two rows are a trade. Adding
verified-healthy tiles in the land cover the model kept misreading cut false alarms from 8.8% to 6.6%
of pixels and cost 0.022 IoU. Recall went up rather than down. I picked lower commission, because a
false alarm on healthy forest becomes a wrong correction handed to a land manager, and that is worse
than a slightly loose boundary on real damage. That is a judgement, not a measurement. Someone
optimising for boundary accuracy could reasonably pick the other row.

## One mistake behind both failure modes

The model had two failures that looked like opposites. It painted damage across healthy open ground,
and it went completely silent under closed canopy on about 14% of damaged crops.

They are one rule seen from two sides. A dead conifer and bare soil are nearly the same colour from
above, and the model settled that ambiguity the cheapest way it could: **pale un-vegetated ground
means damage.**

Rather than assert that from looking at thumbnails, every crop was scored with an excess-green
openness index and compared against performance. The test that carries the weight is on crops
independently verified to hold no damage, where "the label was incomplete" is not available as an
excuse. There, open crops get several times more of their pixels falsely painted than closed ones.

Scoring the training set the same way explained it. Among training tiles more than half bare ground,
there were **61 damage against 12 healthy, 5.1 to 1**. The model had almost never been shown pale open
ground that was not damage. Its rule was an accurate summary of its training data.

The fix needed no new imagery. Ninety-five verified-negative tiles had already been downloaded and were
sitting unused, kept out of training only because nobody had reviewed them. Reviewing them moved the
ratio to 1.68 to 1 and produced the deployed row above.

**The bias is reduced, not removed.** Open crops still get roughly two to three times more of their
pixels falsely painted than closed ones, in all three final runs. I had set a 6% commission target
before running the experiment, and the mean missed it at 6.6%.

**The review also turned up something the metric cannot show.** Going through those tiles by eye,
roughly half of the open-ground tiles the survey had recorded as undamaged held visible standing dead
trees. That points the other way from the metric. If the survey under-detects mortality on open ground,
then some of what I have been measuring as model false alarms may be the model finding real damage and
being scored against an incomplete label. That is a hypothesis and it needs field data, not more
imagery. It is also the most useful thing here to hand back to the survey team.

<aside class="series-link">
  <span class="series-kicker">Deep dive</span>
  <p>The full diagnosis &mdash; including the analysis I got wrong first, and what reviewing those tiles
  turned up about the survey itself &mdash; is in
  <a href="/blog/bare-ground-and-the-survey/">The Survey Missed What the Model Saw</a>.</p>
</aside>

## What does not work yet

1. **Small damage is missed.** About 14% of damaged crops get no prediction at all, and an empty
   prediction scores zero recall. The model has learned a large-area texture cue and cannot resolve
   individual dead crowns. This is now the largest single source of lost IoU.
2. **Commission is 6.6%**, still above the 6% target and still tilted toward open ground.
3. **Labels are partial.** Annotators traced damage clusters, not every tree, so every IoU here is a
   lower bound rather than an unbiased estimate.
4. **No out-of-region test.** Cross-validation is split by geography but stays inside Oregon. Nothing
   here measures transfer to another state. One region should be held out completely and scored once,
   at the end.
5. **The `damage` label is not fully defined.** Whether complete mortality that has become bare ground
   counts as damage is a question for the survey, not the model. It also conflicts directly with the
   rule the model has to learn to avoid false alarms in open woodland.

## Negative results

Each of these cost time and closed a direction, which is why they are written down instead of deleted.

| Tried | Outcome |
|---|---|
| Affine alignment of the polygon | Fails. The shape is wrong, not the position |
| NDVI / NIR as an extra input channel | Hurt, then became a no-op once an initialisation bug was fixed |
| ADS prior as a soft distance-transform hint | Negative across five logged runs |
| Overlapping the negative crops to get more of them | No improvement. More views of the same tiles are not more information |
| Raising the minimum-label threshold to drop thin crops | Lifts IoU while leaving recall flat, so it is a metric artefact. Rejected |
| Gating corrections on model confidence before deployment | Worse at every coverage than correcting everything |

That last one is worth stating the positive way. **Selective deployment is unnecessary.** Holding a
correction back means keeping a polygon worth 0.115, and even the model's weakest crops beat that.

## What shipped

| Folder | Contents |
|---|---|
| [`data_prep/`](https://github.com/musaqlain/ads-forest-damage-segmentation/tree/master/data_prep) | Imagery fetching, geographic balancing, fixed-resolution crop generation |
| [`annotation/`](https://github.com/musaqlain/ads-forest-damage-segmentation/tree/master/annotation) | Labelme helpers, auto-drafting, mask conversion, review and manifest tooling |
| [`training/`](https://github.com/musaqlain/ads-forest-damage-segmentation/tree/master/training) | TreeFinder pretraining and `finetune_30cm.py`, cross-validation and evaluation |
| [`validation/`](https://github.com/musaqlain/ads-forest-damage-segmentation/tree/master/validation) | Independent checks against a second expert's corrections |
| [`studies/`](https://github.com/musaqlain/ads-forest-damage-segmentation/tree/master/studies) | Earlier experiments, including the retired alignment work behind the reshape finding |

Full numbers, every negative result and every caveat:
**[RESULTS.md](https://github.com/musaqlain/ads-forest-damage-segmentation/blob/master/RESULTS.md)**.

## How the evaluation is kept honest

Three rules, because the easy version of this project reports a much better number.

1. **Folds are split by geography.** Nearby tiles look nearly identical, so a random split would let
   the model see its own test data.
2. **Crops from one tile stay together**, so overlapping crops never straddle train and test.
3. **A separate slice picks the settings.** The training epoch and the decision threshold are chosen
   from 240 candidates. Choosing them on the test set inflates the score by about 0.056, so they are
   chosen on 15% held out of the training data instead.

Everything is reported as a mean over repeated runs. Single runs here swing by up to 0.03 IoU, which is
larger than several of the effects being tested.

## What I would do next

1. **Hold out a region completely** and score it once. That is the missing external check.
2. **Attack the silent crops**, now the largest source of lost IoU. Probably a loss-weighting or
   resolution question rather than a data question.
3. **Audit the open-ground negatives against field data.** If the survey really does miss mortality
   there, it changes how every commission number in this project should be read.
4. **Settle the label definition** with the survey team.

## Thanks

To Ben Weinstein and Josh Veitch-Michaelis for mentoring this, and in particular for the push after
the midterm toward growing the right data rather than more of it. That produced the geographic
balancing work, and eventually the open-ground result.

Pretraining uses the TreeFinder dataset (NeurIPS 2025). Imagery from USDA NAIP and Oregon OSIP.
Annotations from the USFS Aerial Detection Survey.

---

*Follow along with the [#gsoc](/tags/gsoc/) tag.*
