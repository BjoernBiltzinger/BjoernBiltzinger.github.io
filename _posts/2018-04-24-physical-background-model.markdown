---
layout: post
title:  A Physical Background Model for the Fermi Gamma-ray Burst Monitor
date:   2020-07-30 00:00:00 +0300
image:  '/images/posts_main/satellite1.png'
tags:   [peer-review]
---
Our research team at the Max-Planck-Institut für Extraterrestrische Physik has developed the first physically motivated background model for the Gamma-ray Burst Monitor (GBM) aboard the Fermi satellite. The GBM, consisting of 14 detectors (12 sodium iodide and 2 bismuth germanate detectors), provides all-sky coverage for detecting transient events like gamma-ray bursts (GRBs) - extremely energetic explosions in distant galaxies.
<div class="gallery-box">
  <div class="gallery">
    <img src="/images/gbm_background/fig1.jpg" alt="GBM detector layout">
  </div>
  <em>Fig. 1: Layout of the Gamma-ray Burst Monitor on the Fermi satellite. The 14 detectors (colored cylinders) are arranged to provide maximum sky coverage. The satellite base (gray), Large Area Telescope (dark blue), and solar panels (light blue) are also shown.</em>
</div>
Traditional methods for analyzing GBM data rely on fitting polynomial functions to estimate the background radiation. However, this approach becomes problematic for long-duration events or when the background varies in complex, non-polynomial ways. Our model addresses these limitations by incorporating the physics of various background sources and the complex detector response.
For analyzing GBM data, understanding how each detector responds to incoming radiation is crucial. The effective area - the detector's ability to detect photons from different directions - varies significantly based on the photon's arrival direction and energy.
<div class="gallery-box">
  <div class="gallery">
    <img src="/images/gbm_background/fig4.jpg" alt="Effective area visualization">
  </div>
  <em>Fig. 4: Effective area seen by a 38 keV photon for different arrival directions around detector n0, shown for four different viewing angles. The red arrow marks the detector's line of sight, the dark cylinder represents the detector housing, and the magenta cylinder shows the detector crystal. The high-sensitivity regions (warmer colors) indicate where the detector is most effective at detecting incoming photons.</em>
</div>
Our physical model accounts for several key background sources:

1. Earth albedo (gamma rays reflected from Earth's atmosphere)
2. Cosmic gamma-ray background (CGB, from unresolved extragalactic sources)
3. Solar contribution
4. Point sources (currently including the Crab Nebula)
5. Effects from the South Atlantic Anomaly (SAA, a region of heightened radiation)
6. Cosmic ray interactions

The model proved particularly valuable for analyzing challenging cases like ultra-long GRB 091024, which showed multiple emission peaks over more than 1000 seconds. Traditional polynomial methods struggle with such long-duration events, but our physics-based approach maintained accuracy throughout the event.
<div class="gallery-box">
  <div class="gallery">
    <img src="/images/gbm_background/fig18.jpg" alt="Ultra-long GRB analysis">
  </div>
  <em>Fig. 18: Analysis of the ultra-long GRB 091024 using detector n0 in two energy ranges (27-50 keV and 102-295 keV). The black points show the observed data, while the red line shows our total model fit. The colored lines below show the individual contributions from different background components: cosmic rays (purple), Earth albedo (blue), cosmic gamma-ray background (light blue), and other sources. The model successfully reveals multiple emission peaks while accurately tracking the varying background.</em>
</div>
The model represents a significant advancement in three key areas:

1. Improved background estimation for spectral analysis and localization of GRBs
2. Potential detection of new transient events, particularly those with slow rise times
3. Better understanding of the individual contributions from different background sources

By building a model based on physical processes rather than empirical fitting, we've created a more robust tool for analyzing GBM data. This approach should help uncover previously undetectable events and advance our understanding of the gamma-ray sky.

Paper Link: https://www.aanda.org/articles/aa/full_html/2020/08/aa37347-19/aa37347-19.html