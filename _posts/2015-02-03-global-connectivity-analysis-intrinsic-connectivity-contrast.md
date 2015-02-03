---
layout: post
title: "A method to identify areas of brain dysconnectivity"
excerpt: "Resting-state functional Magnetic Resonance Imaging (rs-fMRI) has
become a popular tool to study brain disorders. There has been accumulating
evidence showing association of various brain disorders, such as schizophrenia
and autism, with brain dysconnectivity and, in general, with alterations in
brain networks.  In this post, I would like to describe a common method used to
identify areas of dysconnectivity in the brain, namely the global brain
connectivity analysis."
---

Introduction
------------

Resting-state functional Magnetic Resonance Imaging (rs-fMRI) has become a
popular tool to study the brain connectivity and dynamics 
{% cite Sporns:2007 Power:etal:2014 %}. At the same time, there has been
 increasing evidence of an association between various brain disorders, such 
as schizophrenia and autism, and disturbances in brain 
networks {% cite Fornito:Bullmore:2014 Rubinov:Bullmore:2013 %}. 

In this post, I would like to describe a common method which can be used to
compare brain connectivity profiles between groups of subjects and therefore
identify areas of brain dysconnectivity, namely the global brain connectivity
analysis, also abbreviated as GBC {% cite Cole:etal:2010 Cole:etal:2011 %}. Its
main advantages are the following: 

* The method is voxel-based and therefore does not require
any arbitrary definitions of regions-of-interest (ROIs). 
* Moreover, the method works with a weighted measure of voxel
connectivity {% cite Rubinov:Sporns:2011 %}.
This alleviates potential issues connected with thresholding and/or 
binarization of connectivity values. 

Global brain connectivity analysis
----------------------------------

The GBC value of a voxel is its average connectivity strength to all
other voxels in the brain. Its calculation is as follows. For each voxel:

* Compute its correlation to all other voxels in the brain.
* Transform the correlation values to *z*-scores using [Fisher's *r*-to-*z* 
transform](http://en.wikipedia.org/wiki/Fisher_transformation).
* Average the *z*-scores.

The final average value is the GBC value of the given voxel. Repeating this
process, it is possible to calculate GBC maps for each subject in the study.

In order to identify voxels with differing connectivity profiles in
the two groups of subjects, we now need to run a series of two-tailed 
independent samples *t*-tests between subjects in the two groups, 
one test for each voxel. The resulting statistical maps should be 
corrected for [multiple 
comparisons](http://en.wikipedia.org/wiki/Multiple_comparisons_problem).

Examples in the literature and related methods
----------------------------------------------

Since its introduction, the GBC analysis has been used extensively 
to identify areas of altered brain functional connectivity. Examples
include prefrontal cortex dysconnectivity patterns in schizophrenia
{% cite Cole:etal:2011 %}, the effect of ketamine on 
resting-state functional connectivity {% cite Driesen:etal:2013 %},
and dysconnectivity patterns in bipolar disorder
{% cite Anticevic:etal:2013 %}.

There have also been other, similar methods introduced in the literature. 
As an example, we can cite the intrinsic connectivity contrast 
analysis {% cite Martuzzi:etal:2011 %},
which differs in several details (e.g. the way correlation values are
averaged, the possibility of thresholding), but overall uses very similar
ideas.

References
----------

{% bibliography --cited %}
