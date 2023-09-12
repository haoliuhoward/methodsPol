---
title: "DAGs"
citeproc: true
bibliography: ../../static/bib/references.bib
csl: ../../static/bib/chicago-fullnote-bibliography-no-bib.csl
menu: 
  readings:
    parent: Readings
    weight: 8
type: docs
output:
  blogdown::html_page:
    toc: true
editor_options: 
  chunk_output_type: console
---

<script src="/rmarkdown-libs/fitvids/fitvids.min.js"></script>
<script src="/rmarkdown-libs/fitvids/fitvids.min.js"></script>

## Recommended

- work through the examples in the `ggdag` package vignette [here](https://cran.r-project.org/web/packages/ggdag/vignettes/intro-to-dags.html).[^1]

- <i class="fas fa-book"></i> [Directed Acyclical Graphs in Causal Inference: The Mixtape](https://mixtape.scunning.com)[^2]

## Slides

### Tuesday

[Class slides here](/slides/09-dags.html)

<div class="shareagain" style="min-width:300px;margin:1em auto;" data-exeternal="1">
<iframe src="/slides/09-dags.html" width="1600" height="900" style="border:2px solid currentColor;" loading="lazy" allowfullscreen></iframe>
<script>fitvids('.shareagain', {players: 'iframe'});</script>
</div>

### Thursday

[Class slides here](/slides/10-controls-I.html)

<div class="shareagain" style="min-width:300px;margin:1em auto;" data-exeternal="1">
<iframe src="/slides/10-controls-I.html" width="1600" height="900" style="border:2px solid currentColor;" loading="lazy" allowfullscreen></iframe>
<script>fitvids('.shareagain', {players: 'iframe'});</script>
</div>

[^1]: Don’t just copy/paste the code; write it out yourself! Change the variables in the smoking/cancer example to something of interest to you.

[^2]: This reading is mathy-er than what we’ve covered so far. It also features code from STATA, which we don’t use. Don’t worry about that so much; we will focus on broad concepts/ideas.
