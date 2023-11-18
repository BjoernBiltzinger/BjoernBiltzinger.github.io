---
layout: paper
title: Physical Background Model for Fermi/GBM
category: publications
preview_text: The background in low Earth orbit gamma-ray telescopes is complex and changing on a short time scale of ~100 seconds, due to the short orbital period of the satellites and the large number of different sources contributing to the background. We present the first physical background model, which is able to fit the background of several detectors of the gamma-ray burst monitor (GBM) simultaneously, and its applications.
authors: <b>Björn Biltzinger</b>, <a href="https://orcid.org/0000-0002-9424-4385">Felix Kunzweiler</a>, <a href="https://nl.linkedin.com/in/kilian-toelge">Kilan Toelge</a>, <a href="https://orcid.org/0000-0002-9875-426X">Jochen Greiner</a>, <a href="https://orcid.org/0000-0003-3345-9515">J. Michael Burgess</a>
link_to_paper: https://www.aanda.org/articles/aa/full_html/2020/08/aa37347-19/aa37347-19.html
citations: 16
---

## Key Messages

1. It is possible to construct a physical background model for a gamma-ray telescope in an low earth orbit, with proper forward folding techniques
2. The background for a gamma-ray telescope in an low earth orbißt is a superposition of many different sources
3. Especially at high energies (>few hundred keV) the cosmic ray component gets dominant. Further simulation of this component for future missions is needed to improve the background model. 

## Summary

The gamma-ray burst monitor (GBM) is an experiment onboard of the Fermi satellite, which consists of 14 individual detectors, pointing into different directions. This leads to an all-sky coverage, with the exception of the part of the sky that is occulted by the Earth. The orbital of the Fermi satellite is only about 95 minutes, due to the low earth orbit (LEO) of the Fermi satellite (only ~550 km above the ground). This small orbital period and the fact that the magnetic shielding of charged particles (cosmic rays), by the earth magnetic field, varies strongly on the orbital path lead to variations of the background on the timescale of ~ 100 seconds. For the analysis of GBM data it is important to take this background into account. Usually this was done by fitting polynomials to the time before and after the time of interest and assuming that this polynomial is also valid during the time of interest. In this work we developed the first physical background model, that can improve this significantly and lead the path to analyze transients that are too long for the classical polynomial approach.

## Plots

<img src="{{site.baseurl}}/assets/images/gbm_background_full.jpg">
Background for one day, one of the GBM detectors (n8) and one energy channel (102-295 keV). It is interesting to see how many different background components have a significant influence on the background and that they are sometimes changing very quickly. The blue vertical line indicates the time of GRB091024 that is also visible as deviations from the background model fit.
<hr>
