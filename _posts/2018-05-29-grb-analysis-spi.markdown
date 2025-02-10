---
layout: post
title:  Improving Gamma-Ray Burst Analysis with INTEGRAL/SPI
date:   2022-03-25 00:00:00 +0300
image:  '/images/posts_main/satellite2.png'
tags:   [peer-review]
---
Our understanding of gamma-ray bursts (GRBs) in the tens of keV to MeV range has been largely shaped by observations from the Fermi Gamma-ray Burst Monitor (GBM) over the past decade. However, the Spectrometer on INTEGRAL (SPI) offers unique capabilities that have been underutilized - particularly its exceptional energy resolution of ~0.2% at 1.3 MeV compared to GBM's ~10%. In this work, we demonstrate how advanced analysis techniques can maximize SPI's potential for studying GRB emission mechanisms.

<div class="gallery-box">
<div class="gallery">
<img src="/images/spi_grb/fig3.jpg" alt="GRB light curve comparison">
</div>
<em>Fig. 3: Light curve of GRB 120711A in one GBM detector (bottom) and one SPI detector (top). The green shaded areas mark the background intervals, while the blue shaded area indicates the analyzed active time interval.</em>
</div>

To leverage SPI's capabilities, we developed PySPI - a new open-source analysis framework that implements proper forward-folding spectral analysis. This approach fully accounts for the instrument response and background, yielding more precise constraints than previous methods.

<div class="gallery-box">
<div class="gallery">
<img src="/images/spi_grb/fig8.jpg" alt="Model posterior plots">
</div>
<em>Fig. 8: Model posterior plots showing the results of our physical synchrotron model fit to the data of GRB 120711A. Left panel shows SPI results compared to the combined fit, while the right panel shows GBM results compared to the combined fit. The combined analysis reduces the allowed parameter space.</em>
</div>

We validate our approach using GRB 120711A as a test case. The results show that PySPI provides significantly tighter parameter constraints compared to standard analysis tools. More importantly, when combining SPI and GBM data, we demonstrate that both instruments' spectra can be well-fit with a physical synchrotron emission model.

<div class="gallery-box">
<div class="gallery">
<img src="/images/spi_grb/fig9.jpg" alt="Data and best-fit model comparison">
</div>
<em>Fig. 9: Data and best-fit model in count space for three SPI detectors and GBM detectors (NaI and BGO). The excellent agreement across both instruments demonstrates the validity of our physical synchrotron model.</em>
</div>

This work opens new possibilities for studying GRB emission mechanisms by fully utilizing SPI's exceptional spectral resolution. The combined analysis with GBM data provides unprecedented constraints on physical emission models, helping to resolve long-standing questions about the origin of prompt GRB emission.

Paper Link: https://www.aanda.org/articles/aa/full_html/2022/07/aa43189-22/aa43189-22.html