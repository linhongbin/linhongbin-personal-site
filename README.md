# hongbinlin.com

Source for the personal site of **Hongbin (Ben) Lin** — robotics manipulation, force control,
and embodied VLA. The site is live at <https://hongbinlin.com>.

Static site built with [Jekyll](https://jekyllrb.com/) and served by GitHub Pages. Every push to
`master` rebuilds and republishes the live site.

```
bundle exec jekyll serve
```

## What is on the site

- **Home** — a short bio, selected writing, and the publication list (`index.html`, with the
  entries kept in `_data/`).
- **Blog** — build notes from an open benchmark for force-sensing contact-rich manipulation:
  measurements, rig decisions, and the reasoning behind them, written as they happen
  (`blog.html`, posts in `_posts/`).
- **CV** — a PDF, linked from the home page.

## Looking for collaborators

I am building an open benchmark for force-sensing contact-rich manipulation — a graded corpus of
industrial threaded fasteners, a force-data metrology spec, and a pre-registered ablation on
whether a real force sensor beats a joint-current estimate. Full rig under ¥30k; data and spec
CC-BY-4.0, code Apache-2.0.

1. **Force estimate vs. ground truth** — a pre-registered three-way ablation on whether a 6-axis
   force/torque sensor actually beats a joint-current estimate for contact-rich policies.
2. **A graded corpus of seized fasteners** — industrial threaded joints graded by difficulty:
   infinitely resettable, cheap to run, and failure modes that a camera cannot see but a wrench
   signal can.
3. **Force-data metrology** — a spec for what makes a force dataset trustworthy: calibration
   traceability, sampling rate, units, and provenance.

I would like to hear from you if you work on force or tactile policies, contact-rich benchmarks,
or force-data quality — or if you have access to industrial fastening or disassembly data or
hardware.

The benchmark repository is not public yet, so there is nothing to link to here. Reach me at
**hongbinlin@link.cuhk.edu.hk** and I will share the current state of the corpus, the metrology
spec draft, and the ablation protocol directly.

## License

The site template is derived from Keunhong Park's Jekyll template
([keunhong/keunhong.github.io](https://github.com/keunhong/keunhong.github.io)) and is licensed as below.
The licence covers the template — layout, styles, and structure — not the prose, figures, or data
published through it.

<a rel="license" href="http://creativecommons.org/licenses/by-sa/4.0/"><img alt="Creative Commons License" style="border-width:0" src="https://i.creativecommons.org/l/by-sa/4.0/88x31.png" /></a><br />This work is licensed under a <a rel="license" href="http://creativecommons.org/licenses/by-sa/4.0/">Creative Commons Attribution-ShareAlike 4.0 International License</a>.

Intended licensing for the benchmark itself, once it is released: dataset and metrology spec under
CC BY 4.0, code under Apache 2.0. Those terms are stated here as intent; the authoritative license
files will ship with the benchmark release.
