---
layout: paper
title: Automatic detection of long-duration transients in Fermi-GBM data
category: publications
preview_text: Using the physical background model we developed and a change point detection algorithm to identify long but slowly rising transients (e.g. ultra long gamma-ray bursts) in the GBM data.
authors: <a href="https://orcid.org/0000-0002-9424-4385">Felix Kunzweiler</a>, <b>Björn Biltzinger</b>, <a href="https://orcid.org/0000-0002-9875-426X">Jochen Greiner</a>, <a href="https://orcid.org/0000-0003-3345-9515">J. Michael Burgess</a>
link_to_paper: https://www.aanda.org/articles/aa/full_html/2022/09/aa43287-22/aa43287-22.html
citations: 3
---

## Key Messages

1. Our physical background model can be used to identify untriggered transients in the Fermi/GBM data
2. The high-dimensional change-point detection algorithm used here results in a near-optimal detection for long-duration signals even for fluxes down to 0.7 Crab.
3. Successfully triggered on a flaring outburst of the Vela X-1 pulsar. Were able to confirm the known spin period of ~280 seconds of the neutron star with Fermi/GBM data.

## Summary
<i>Context:</i> In the era of time-domain, multi-messenger astronomy, the detection of transient events on the high-energy electromagnetic sky has become more important than ever. Previous attempts to systematically search for onboard, untriggered events in the data of Fermi-GBM have been limited to short-duration signals with variability time scales smaller than ≈1 min. This is due to the dominance of background variations on longer timescales.

<i>Aims:</i> In this study, we aim to achieve a detection of slowly rising or long-duration transient events with high sensitivity and a full coverage of the GBM spectrum.

<i>Methods:</i> We made use of our earlier developed physical background model, which allows us to effectively decouple the signal from long-duration transient sources from the complex varying background seen with the Fermi-GBM instrument. We implemented a novel trigger algorithm to detect signals in the variations of the time series that is composed of simultaneous measures in the light curves of the different Fermi-GBM detectors in different energy bands. To allow for a continuous search in the data stream of the satellite, the new detection algorithm was embedded in a fully automatic data analysis pipeline. After the detection of a new transient source, we also performed a joint fit for spectrum and location using the BALROG algorithm.

<i>Results:</i> The results from extensive simulations demonstrate that the developed trigger algorithm is sensitive down to sub-Crab intensities (depending on the search timescale) and has a near-optimal detection performance. During a two month test run on real Fermi-GBM data, the pipeline detected more than 300 untriggered transient signals. We verified, for one of these transient detections, that it originated from a known astrophysical source, namely, the Vela X-1 pulsar, showing pulsed emission for more than seven hours. More generally, this method enables a systematic search for weak or long-duration transients.

## Plots
<img src="{{site.baseurl}}/assets/images/detection_prob.jpg">
Significance of sources with different strengths for different integration time. A significance of 5 sigma would be a detection.
<hr>
<img src="{{site.baseurl}}/assets/images/flaring_signal.jpg">
Data in a few different detectors and energy channels. The flaring signal above the background is clearly visible.
<hr>
