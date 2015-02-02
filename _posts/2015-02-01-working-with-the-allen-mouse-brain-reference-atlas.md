---
layout: post
title: "Working with the Allen Mouse Brain Reference Atlas"
excerpt: "The Allen Brain Atlas portal is an entry point to a wealth of
publicly available information on the structure, function, and development of
the nervous system. In the following series of posts, I would like to introduce
several of those resources and show how to work with them."
---

Introduction
------------

The [Allen Institute for Brain Science](http://alleninstitute.org/) has been
producing a wealth of information on the structure, function, 
and development of the nervous system, ranging from atlases to gene expression
maps for the mouse, human, and non-human primate. All these data sets are
publicly available on the [Allen Brain Atlas 
portal](http://www.brain-map.org/), and in the following series of posts,
I would like to introduce the resources available for researchers interested
in the mouse brain. These include:

* the [Allen Mouse Brain Reference Atlas](http://atlas.brain-map.org/),
* the [Mouse Brain Atlas](http://mouse.brain-map.org/),
* the [Developing Mouse Brain Atlas](http://developingmouse.brain-map.org/),
* the [Mouse Connectivity Atlas](http://connectivity.brain-map.org/), and
* the [Mouse Spinal Cord Atlas](http://mousespinal.brain-map.org/).

The Allen Mouse Brain Reference Atlas has been originally introduced within the Mouse Brain
Atlas project, to support the development of a genome-wide atlas of gene
expression in the adult mouse brain {% cite Lein:etal:2007 %}. It is a brain
reference atlas based on annotated [Nissl
sections](http://openwetware.org/wiki/Nissl_staining) of the mouse brain, and
its development has been described in two technical white papers published by
the Allen Institute for Brain Science (see the
[2008](http://help.brain-map.org/download/attachments/2818169/AllenReferenceAtlas_v1_2008_102011.pdf?version=1)
and
[2011](http://help.brain-map.org/download/attachments/2818169/AllenReferenceAtlas_v2_2011.pdf?version=1)
versions).

Prerequisites
-------------

This tutorial assumes that the reader has some basic knowledge of [Python
programming](https://docs.python.org/2/tutorial/). As we will be working with
large collections of numbers, some knowledge of [NumPy](http://www.numpy.org/),
the Python package for scientific programming, will also be useful. We will
use [NIPY](http://nipy.org/), and more specifically
[NiBabel](http://nipy.org/nibabel), to work with neuroimaging data and file
formats, and [matplotlib](http://matplotlib.org/) to produce figures; however,
no advanced knowledge is required.

I recommend using the [IPython
Notebook](http://ipython.org/notebook.html) computational environment for your
experiments.

Downloading the atlas
---------------------

While it is possible to browse the Allen Reference Atlas
[online](http://atlas.brain-map.org/), it might be convenient to have a
copy of it on your own computer and in your favourite format, e.g. in the
[NIfTI format](http://brainder.org/2012/09/23/the-nifti-file-format/). The
portal does not make the atlas available directly in the NIfTI format; however,
all volumetric data files describing the brain volume reconstructed from Nissl
sections and the structure annotations that we will need are available for
download on the [API
site](http://help.brain-map.org//display/mousebrain/API#API-3DReferenceModels)
of the atlas.

Reading the atlas in Python
---------------------------

First, we need to download the grayscale Nissl volume from the atlas API site.
This file, which is called "atlasVolume" (download the zip
[here](http://api.brain-map.org/api/v2/well_known_file_download/113567585),
29.1 MB), is a binary file containing the 3-D volumetric data (described by
three indexes: x, y, and z) packed into a 1-D numerical array (described by
just one index).  The data are packed in the Fortran-like index order, in which
the first index changes fastest, and the last index changes slowest (for 2-D
matrices, this type of packing is also referred to as [column-major
order](http://en.wikipedia.org/wiki/Row-major_order)).  For an illustration,
see the [API
site](http://help.brain-map.org//display/mousebrain/API#API-3DReferenceModels)
of the atlas. 

To read the extracted binary file, we use the
[numpy.fromstring](http://docs.scipy.org/doc/numpy/reference/generated/numpy.fromstring.html)
method:
{% highlight python %}
import numpy as np

volumepath = 'path/to/atlasVolume.raw'
with open(volumepath, "rb") as f:
    volume = np.fromstring(f.read(), dtype='uint8')
{% endhighlight %} 

This returns a 1-D array intialized from the binary file. We need to unpack
this array and reshape it to a 3-D array of the grayscale Nissl volume. This
is done using [numpy.reshape](http://docs.scipy.org/doc/numpy/reference/generated/numpy.reshape.html):
{% highlight python %}
dim = [528, 320, 456]    
volume = np.reshape(volume, dim, order='F')
{% endhighlight %}
Notice the order parameter, which specifies that the elements were packed using
Fortran-like index order.

Displaying sections of the volume
---------------------------------

As explained on the API site of the atlas, the reference space is in such an
orientation, in which the x axis represents the anterior-to-posterior
direction, the y axis the superior-to-inferior direction, and the z axis the
left-to-right direction.

We can therefore display a coronal section of the volume within IPython
Notebook using the
[matplotlib.pyplot.imshow](http://matplotlib.org/api/pyplot_api.html#matplotlib.pyplot.imshow)
function, by fixing the x axis (anterior-to-posterior) value:
{% highlight python %}
import matplotlib.pyplot as plt
%matplotlib inline

#coronal section
plt.imshow(volume[263,:, :], cmap = plt.cm.Greys_r)
{% endhighlight %}
This should result in an image shown below:

![Allen Reference Atlas, coronal section](/downloads/images/allen_reference_atlas_coronal_section.png)

Similarly, a horizontal section can be displayed using the following code, in 
which we fix the y axis (superior-to-inferior) value:
{% highlight python %}
#horizontal section
plt.imshow(volume[:,200,:], cmap = plt.cm.Greys_r)
{% endhighlight %}

Saving the volume in the NIfTI format
-------------------------------------

To save the volume as a NIfTI file, we will use the
[NiBabel](http://nipy.org/nibabel) package. I recommend reading the brief
[NiBabel manual](http://nipy.org/nibabel/manual.html) in case you aren't
familiar with this package. Moreover, if you aren't familiar with how the
relationship between voxel coordinates and the world coordinate system is
defined and encoded in a NIfTI file, have a look at the "Coordinate systems and
affines" [tutorial](http://nipy.org/nibabel/coordinate_systems.html).

In the following code snippet, we first create a NIfTI header, in which we
define the voxel dimensions (zooms) and the relationship of voxel coordinates
to the world coordinate system (the sform matrix). Note that NIfTI assumes the
world coordinate system to be in the RAS orientation: the x axis represents the
left-to-right direction, the y axis the posterior-to-anterior direction, and
the z axis the inferior-to-superior direction, which is different from
the orientation in which the volumetric data are encoded in the Allen
Mouse Brain Reference Atlas. We then create a NIfTI image by
combining the volumetric data with the header and save it in the outputpath
location:

{% highlight python %}
import nibabel as nib

h = nib.Nifti1Header()
h.set_data_shape(volume.shape)
h.set_zooms([0.025, 0.025, 0.025])
h.set_sform([[0,0,1,0],[-1,0,0,0],[0,-1,0,0]], code='aligned')
h.set_data_dtype(np.uint8)

f = nib.Nifti1Image(volume, None, header=h)

outputpath = 'path/to/output/allen_mouse_brain_reference_atlas.nii.gz'
nib.save(f, outputpath)
{% endhighlight %}

The resulting grayscale Nissl volume used in the Allen Mouse Brain Reference
Atlas can be downloaded in the NIfTI format
[here](/downloads/other/allen_mouse_brain_reference_atlas.nii.gz) (29.9 MB).

In the next post, we will work with structural annotations. You can follow the
series by subscribing to the RSS feed below. If you have any comments,
questions or suggestions, don't hesitate to contact me at adam AT adliska.com.

References
----------

{% bibliography --cited %}
