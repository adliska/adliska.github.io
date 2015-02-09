---
layout: post
title: Global brain connectivity analysis in AFNI
excerpt: "In the last post, I talked about the global brain connectivity
analysis and its use in identifying areas of differing brain connectivity
across two or more conditions. In this post, I show how to implement
this method using the AFNI set of neuroimaging data analysis tools."
---

Introduction
------------
In the [last post]({% post_url 2015-02-03-global-connectivity-analysis-intrinsic-connectivity-contrast %}),
I talked about the global brain connectivity (GBC)
analysis and its use in identifying areas of differing brain connectivity
across two or more conditions {% cite Cole:etal:2010 Cole:etal:2011 %}. 
In this post, I show how to implement this method using the AFNI set of 
neuroimaging data analysis tools 
([website](http://afni.nimh.nih.gov/afni/), 
[wiki](http://en.wikipedia.org/wiki/Analysis_of_Functional_NeuroImages)).

As a quick reminder, the GBC analysis consists in: (1) computing 
the GBC voxel maps for each subject, and (2) comparing these maps across
groups. For more details, check out the [previous 
post]({% post_url 2015-02-03-global-connectivity-analysis-intrinsic-connectivity-contrast %}).

Computing global brain connectivity maps
----------------------------------------

The GBC value of a voxel is computed in the following way:

* Compute its correlation to all other voxels in the brain (or in the
provided voxel mask).
* Transform the correlation values to *z*-scores using [Fisher's *r*-to-*z* 
transform](http://en.wikipedia.org/wiki/Fisher_transformation).
* Average the *z*-scores.

As a matter of fact, AFNI already provides a simple way to calculate just 
that, namely the [`3dTcorrMap`](http://afni.nimh.nih.gov/pub/dist/doc/program_help/3dTcorrMap.html),
which, according to its documentation:

> For each voxel time series, computes the correlation between it
> and all other voxels, and combines this set of values into the
> output dataset(s) in some way.

The combination of "some way" in our case is a simple mean of 
*z*-transformed correlation values, which is accomplished using the
`-Zmean` option:

> -Zmean pp = Save tanh of mean arctanh(correlation) into 'pp'

The calculation of a subject GBC map therefore boils down to running the 
following command:

{% highlight bash %}
3dTcorrMap \
    -input $timeseries \
    -mask $voxelmask \
    -Zmean ${name}_gbc_r.nii.gz \
    &> log_${name}.txt
{% endhighlight %} 

As the `-Zmean` option transforms the GBC values back to *r*-values, we
need to convert them back to *z*-scores using 
[`3dcalc`](http://afni.nimh.nih.gov/pub/dist/doc/program_help/3dcalc.html):
{% highlight bash %}
3dcalc \
    -a ${name}_gbc_r.nii.gz \
    -expr "atanh(a)" \
    -prefix ${name}_gbc_z.nii.gz
{% endhighlight %}

Analyzing between-group differences
-----------------------------------

Let's assume that our goal is to discover how brain networks
are affected by disease and we decide to use the GBC analysis
to identify areas of differing brain connectivity.
In this setup, we have two sets of rs-fMRI scans: from control
participants and from patients. We calculate GBC maps for all subjects
in the experiment and then run a series of *t*-tests, one for each voxel,
using AFNI's [`3dttest++`](http://afni.nimh.nih.gov/pub/dist/doc/program_help/3dttest++.html):

{% highlight bash %}
3dttest++ \
    -setA group_one/*_gbc_z.nii.gz \
    -setB group_two/*_gbc_z.nii.gz \
    -mask $voxelmask \
    -prefix gbc_results.nii.gz
{% endhighlight %}

The output file, `gbc_results.nii.gz`, is composed of multiple "sub-bricks",
and it might be helpful to extract the two most important ones into separate files:
{% highlight bash %}
3dcalc \
    -a "gbc_results.nii.gz[0]" \
    -expr "a" \
    -prefix gbc_groupdiff.nii.gz

3dcalc \
    -a "gbc_results.nii.gz[1]" \
    -expr "a" \
    -prefix gbc_tstat.nii.gz
{% endhighlight %}

As the last step of the GBC analysis, the output statistical map
should be subjected to multiple comparisons correction.

References
----------

{% bibliography --cited %}
