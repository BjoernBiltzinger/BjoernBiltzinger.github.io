---
layout: post
title:  Time-resolved spectral catalogue of INTEGRAL/SPI gamma-ray bursts
date:   2023-05-31 15:01:35 +0300
image:  '/images/posts_main/grb1.png'
tags:   [peer-review]
---
Since its launch in 2002, the SPectrometer on INTEGRAL (SPI) has detected hundreds of gamma-ray bursts (GRBs) in the energy range from 20 keV to 8 MeV. While previous studies focused mainly on individual bursts, we present the first comprehensive time-resolved spectral catalog of all GRBs detected by SPI through 2021. The catalog leverages SPI's exceptional energy resolution of ~0.2% at 1.3 MeV - about 100 times better than Fermi/GBM - to provide unique insights into GRB emission mechanisms.

<div class="gallery-box">
<div class="gallery">
<img src="/images/spi_catalog/fig1.png" alt="Effective area comparison">
</div>
<em>Fig. 1: Comparison of the total effective area of GBM and SPI over the SPI energy range. While GBM has better high-energy coverage thanks to its BGO detectors, SPI provides superior spectral resolution in the key 100 keV - 1 MeV range.</em>
</div>

Using our developed PySPI software package, we performed time-resolved spectroscopy of 49 GRBs with 140 distinct time intervals. For each interval, we fit both empirical models (Band function, cutoff power law) and a physical synchrotron emission model. The analysis shows that SPI can provide parameter constraints comparable to GBM for bright bursts, with the distributions of spectral parameters matching well between instruments.

<div class="gallery-box">
<div class="gallery">
<img src="/images/spi_catalog/fig4.png" alt="Time evolution analysis">
</div>
<em>Fig. 5: Observed data in the non-PSD energy range of detector 0, over-plotted with the best fit realisation of the synchrotron model for the brightest time bin in our sample (GRB181201A) (top row) and resulting posterior distribution of the vFv spectrum of the GRB (bottom row).</em>
</div>

A key finding is that the physical synchrotron model can successfully fit the SPI data, with many bursts showing evidence for both fast and slow cooling regimes. The identification of slow cooling periods challenges standard relativistic shock models and suggests alternative particle acceleration mechanisms may be at work.

This catalog demonstrates SPI's capabilities for detailed spectral studies of GRBs and establishes it as a valuable complement to other gamma-ray instruments. The combination of high spectral resolution and broad energy coverage makes SPI particularly well-suited for testing physical emission models and advancing our understanding of the still-mysterious GRB prompt emission mechanism.

Paper Link: https://www.aanda.org/articles/aa/full_html/2023/07/aa45191-22/aa45191-22.html