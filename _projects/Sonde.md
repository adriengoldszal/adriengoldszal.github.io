---
layout: page
title: Atmospheric Balloon
description: Ecole Polytechnique second year scientific research project
img: assets/img/prototype1.png
importance: 3
category: Engineering
---

The physics of the atmosphere, and the electrical circuit it constitutes, are active research topics. More specifically, the conductivity and electric field of the atmosphere are the subjects of theoretical models and experimental measurements. Atmospheric physicists are interested in thunderstorms, the phenomena they generate, and their impact on the global electrical circuit of the atmosphere. Research on the role of more complex discharge phenomena such as Transient Luminous Events and Terrestrial Gamma Ray Flashes is also very important, as well as electromagnetic waves like Schumann resonances.

The use of atmospheric balloons is particularly well-suited for studying these physical phenomena. They allow for the exploration of an altitude range where most of the mechanisms responsible for atmospheric electricity and the direct causes of many associated phenomena occur. Two French research programs using balloons are currently being conducted under the auspices of CNES: the general STRATEOLE-2 program and the STRATELEC program.

Our project follows the continuity of the STRATEOLE-2 and STRATELEC programs, in partnership with the Atmosphères, Observations Spatiales (LATMOS) laboratory, and aims to test and develop a prototype of a gondola, as well as its instrumentation, for use in certain CNES missions.

A detailed report can be found [here](../../assets/pdf/Rapport_final_PSC.pdf)

## Measurements 

**Electric field** The measurement of the three components of the atmospheric electric field, both AC and DC, represents the "baseline" measurement of atmospheric electricity. The objective is to measure the intensity and direction of the field, the altitude profile, the disturbances caused by clouds, and to allow comparison with the conductivity profile. The proposed method relies on the use of four spherical electrodes as indicated in the following diagram:

Three electrodes in the XY plane are used to measure the horizontal components of the electric field. These are connected to the gondola by 1-meter-long fiberglass arms. A final electrode measures the vertical component of the electric field.

The issue of the gondola's rotation was raised, as it helps to correct measurement differences due to the potential surface differences of the electrodes. Even though these differences are potentially small, on the order of a few hundred millivolts, they could play a non-negligible role, as the horizontal electric field is also weak.

<div class="row">
    <div class="col-sm mt-3 mt-md-0 d-flex justify-content-center">
        {% include figure.liquid loading="eager" path="assets/img/prototype1.png" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    Structure of the electrodes (In collaboration with Jean-Jacques Berthelier, LATMOS)
</div>

**Conductivity** To measure the conductivity of the atmosphere, a relaxation method can be used. One of the electrodes is displaced from its equilibrium potential at regular intervals. The conductivity is related to the time it takes for the electrode to return to its equilibrium potential, which will be measured. The measurement of conductivity in this way is interesting because, in addition to the fact that few measurements have been made so far, it would allow us to observe if there are any local variations during cloud crossings or due to urban pollution.

Let us seek a relationship between the relaxation time of a charged conductor and the conductivity of the surrounding medium. In an isotropic conductive medium, Ohm's law, along with charge conservation and the Maxwell-Gauss equation, allows us to obtain:
$$
\rho = \rho_0 e^{-\frac{\sigma t}{\epsilon_0}}
$$

Thus, we obtain the time constant: $$\tau = \frac{\epsilon_0}{\sigma}$$, with $$\sigma$$ is the conductivity of the atmosphere in $$\Omega^{-1}, \text{m}^{-1}$$, and $$\epsilon_0 $$ is the dielectric permittivity of free space.

Considering a spherical electrode at the end of an insulating arm, with a charge $$Q$$, the electric field at a distance much greater than $$R$$ (the radius of the sphere) is given by: $$ E = \frac{Q}{4 \pi \epsilon_0 R^2}$$. Using Ohm's Law and the previous result, we get, for an excitation by a potential $$V_0$$ of the electrode at $$ t = 0$$ produces a potential for $$ t > 0 $$ of:

$$
V = (V_0 - V_{\text{eq}}) e^{-\frac{\sigma t}{\epsilon_0}} + V_{\text{eq}}
$$

This allows us to obtain the conductivity by measuring the relaxation time relative to the potential $$V_0$$ applied to the electrode until it reaches equilibrium $$V_{\text{eq}} $$.

The role of dust in the atmosphere, its electrification, and its consequences on lightning triggering mechanisms, meteorology, and the transport of particles over long distances are still poorly understood \cite{ref1}. 

**Charged aerosols** Electrified dust, particularly during episodes such as those periodically observed, bringing particles from the Sahara Desert for example, generate pulses upon impact with the electrodes. It is important to measure this impact, whose amplitude is proportional to the charge, in order to better understand these phenomena.

**Schumann resonances and other EMR** The measurement of Schumann resonances, as well as other signals generated by lightning or ground-based emitters, is a way to verify the correct operation of the measurement chain.

## Mechanical structure of the gondola

## Electronics and Instrumentation

The main challenge of this project lies in the development of an instrumentation capable of:  
- Adapting to the range of electrostatic potentials in the stratosphere (up to several hundred volts) and its disturbances, particularly those of meteorological origin.  
- Successfully discriminating signals of lower amplitude (Schumann resonances, medium electrostatic and electromagnetic fields, and atmospheric disturbances).  
- Finally, processing these signals efficiently in digital form with sufficient precision to highlight the aforementioned phenomena.

The embedded electronics are therefore made up of three stages:  
- High voltage analog (Pre-amplifier)  
- Low voltage analog filtering  
- Low voltage digital processing

While the voltage of the high voltage stage is mainly determined by the external potential, the low voltage corresponds to an amplitude of ±5V. The entire electronics are powered by a battery of the same amplitude (±5V), and the high voltage circuit is powered via a controlled gain converter that we have built. The gain control is crucial to adapt the power supply of the first stage to the external electrostatic conditions.

<div class="row">
    <div class="col-md-6">
        {% include figure.liquid loading="eager" path="assets/img/filtrage.png" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
    <div class="col-md-6">
        <p>The measurement organization is as follows:</p>  
        <ul>
            <li>In addition to the measurements of the DC field (0-3Hz), as well as those corresponding to Ultra Low Frequencies (ULF: 3Hz-45Hz, Schumann resonances) and Very Low Frequencies (VLF: 45Hz - 2kHz, lightning-associated signals), the following are also included:</li>
            <li>A magnetometer that allows for the orientation of the nacelle, and thus the components of the AC and DC fields in a reference frame linked to the Earth.</li>
            <li>A GPS probe that provides the altitude of the nacelle and allows it to be tracked during the flight for safety purposes.</li>
            <li>Three-axis accelerometers added to the arms to measure parasitic vibrations, which could interfere with the measurement of Schumann resonances.</li>
        </ul>
    </div>
</div>

### The preamplifier 



<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/1.jpg" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/3.jpg" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/5.jpg" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    Caption photos easily. On the left, a road goes through a tunnel. Middle, leaves artistically fall in a hipster photoshoot. Right, in another hipster photoshoot, a lumberjack grasps a handful of pine needles.
</div>
<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/5.jpg" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    This image can also have a caption. It's like magic.
</div>

You can also put regular text between your rows of images.
Say you wanted to write a little bit about your project before you posted the rest of the images.
You describe how you toiled, sweated, _bled_ for your project, and then... you reveal its glory in the next row of images.

<div class="row justify-content-sm-center">
    <div class="col-sm-8 mt-3 mt-md-0">
        {% include figure.liquid path="assets/img/6.jpg" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
    <div class="col-sm-4 mt-3 mt-md-0">
        {% include figure.liquid path="assets/img/11.jpg" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    You can also have artistically styled 2/3 + 1/3 images, like these.
</div>

The code is simple.
Just wrap your images with `<div class="col-sm">` and place them inside `<div class="row">` (read more about the <a href="https://getbootstrap.com/docs/4.4/layout/grid/">Bootstrap Grid</a> system).
To make images responsive, add `img-fluid` class to each; for rounded corners and shadows use `rounded` and `z-depth-1` classes.
Here's the code for the last row of images above:

{% raw %}

```html
<div class="row justify-content-sm-center">
  <div class="col-sm-8 mt-3 mt-md-0">
    {% include figure.liquid path="assets/img/6.jpg" title="example image" class="img-fluid rounded z-depth-1" %}
  </div>
  <div class="col-sm-4 mt-3 mt-md-0">
    {% include figure.liquid path="assets/img/11.jpg" title="example image" class="img-fluid rounded z-depth-1" %}
  </div>
</div>
```

{% endraw %}
