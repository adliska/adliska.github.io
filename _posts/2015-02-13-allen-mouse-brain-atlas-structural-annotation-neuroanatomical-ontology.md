---
layout: post
title: "Exploring the neuroanatomical ontology of the Allen Mouse Brain Atlas"
excerpt: "In an earlier post, I described how to download the Allen Mouse
    Brain Atlas and convert it to the NIfTI format. In this post, we'll have
    a closer look at the structural annotation and the associated 
    neuroanatomical ontology that both come with the atlas."
---

Introduction
------------

In an [earlier post]({% post_url 2015-02-01-working-with-the-allen-mouse-brain-reference-atlas %}),
I described how to download the [Allen Mouse Brain Reference 
Atlas](http://atlas.brain-map.org/) and convert it to the NIfTI format. In case
you missed it, you can download the grayscale Nissl volume of the reconstructed
brain [here](/downloads/other/allen_mouse_brain_reference_atlas.nii.gz) 
(29.9 MB). This volume will serve as a starting point in this post, in which
we'll have a closer look at the structural annotation and the 
associated neuroanatomical ontology.

Hierarchical organization of mouse brain structures
---------------------------------------------------

The Allen Mouse Brain Reference Atlas comes with a neuroanatomical ontology
of brain structures organized in a [hierarchical 
tree](http://en.wikipedia.org/wiki/Tree_structure). Its construction,
nomenclature choices, subsequent changes and other issues were described
in the [2008](http://help.brain-map.org/download/attachments/2818169/AllenReferenceAtlas_v1_2008_102011.pdf?version=1)
 and [2011](http://help.brain-map.org/download/attachments/2818169/AllenReferenceAtlas_v2_2011.pdf?version=1)
technical white papers. However, the best way how to
explore the brain structure hierarchy is using the online [Interactive Atlas 
Viewer](http://atlas.brain-map.org/), in which you can navigate through the 
hierarchy using the panel on the left.

Notice that there are four groups of structures at the highest level of the
organization. These are: (1) grey matter, (2) fiber tracts, (3) ventricular 
systems, and (4) grooves. The grey matter group is further subdivided into
(1.1) cerebrum, (1.2) brain stem, and (1.3) cerebellum. These are subdivided
further, and so on and so forth.

Downloading and reading the structural annotation volume
--------------------------------------------------------

The structural annotation data can be downloaded from the [API 
site](http://help.brain-map.org//display/mousebrain/API#API-3DReferenceModels)
of the reference atlas or directly 
[here](http://api.brain-map.org/api/v2/well_known_file_download/197642854)
(6.7 MB). It is an archive containing two files: `annotation.mhd` and
`annotation.raw`. We will only need the latter, which contains the 3-D 
structure annotation volume packed into a 1-D numerical array. The following 
method `read_annotations()` reads the binary file and transforms it into a 
well-shaped volume. The code is identical to the one used to read the grayscale
Nissl volume and was therefore explained in detail in the [last 
post]({% post_url 2015-02-01-working-with-the-allen-mouse-brain-reference-atlas %}#reading-the-atlas-in-python).

{% highlight python %}
import numpy as np

def read_annotations():
    annotationfile = '/path/to/annotation.raw'
    dim = [528, 320, 456]

    with open(annotationfile, "rb") as f:
        values = np.fromstring(f.read(), dtype='uint32')
    
    values = np.reshape(values, dim, order='F')
    return values
    
{% endhighlight %}

The numerical value of a given voxel represents the structure ID of the 
finest level structure annotated for the voxel. This means, for example,
that in order to select all striatum voxels, we will need to search
for voxels with IDs corresponding to the following structures: STRd (striatum 
dorsal region), STRv (striatum ventral region), LSX (lateral septal 
complex) and sAMY (striatum-like amygdalar nuclei), most of which are
subdivided even further. This is where the ontology comes into play.

Downloading and reading the neuroanatomical ontology
----------------------------------------------------

The ontology can be downloaded as a hierarchically structured [JSON 
file](http://en.wikipedia.org/wiki/JSON); the details are described
on the Allen Brain Atlas API site [Atlas Drawing and 
Ontologies](http://help.brain-map.org/display/api/Atlas+Drawings+and+Ontologies).
As it is not very large, we will call it directly from within our code:

{% highlight python %}
import json
import requests

def read_ontology():
    ontologyurl = 'http://api.brain-map.org/api/v2/structure_graph_download/1.json'
    ontology = json.loads(requests.get(ontologyurl).text)
    
    structures = process_ontology(ontology['msg'])

    return structures

{% endhighlight %}

The goal is to create a [dictionary](http://en.wikipedia.org/wiki/Associative_array)
of brain structures, with the key being the ID of the given brain structure
and the value its associated metadata (name, acronym, parent structure, etc.).
This can be achieved using the following method, which as an input parameter
takes a list of elements from the JSON file (each corresponding to a 
brain structure), processes them one by one and then 
[recursively](http://en.wikipedia.org/wiki/Recursion_(computer_science))
calls itself to process their children structures:

{% highlight python %}
import itertools

def process_ontology(elementlist):
    structures = {}
    
    for element in elementlist:
        structures[element['id']] = {
            'id': element['id'],
            'atlas_id': element['atlas_id'],
            'ontology_id': element['ontology_id'],
            'acronym': element['acronym'],
            'name': element['name'],
            'color_hex_triplet': element['color_hex_triplet'],
            'graph_order': element['graph_order'],
            'st_level': element['st_level'],
            'hemisphere_id': element['hemisphere_id'],
            'parent_structure_id': element['parent_structure_id'],
            'children_structure_ids': [el['id'] for el in element['children']]
        }
        
        if element['children']:
            children = process_ontology(element['children'])
            structures.update(children)
            structures[element['id']]['annotation_ids'] = list(
                itertools.chain(*[children[child]['annotation_ids'] for child in children]))
        else:
            structures[element['id']]['annotation_ids'] = [element['id']]

    return structures
{% endhighlight %}

To generate the dictionary of neuroanatomical structures, we need to
run the following line of code:

{% highlight python %}
structures = read_ontology()
{% endhighlight %}

We can then query the ontology for information associated with individual
regions. For instance, let's have a look at the information about
the prelimbic area, ID 972 (`structures[972]`):

{% highlight javascript %}
{'acronym': u'PL',
 'annotation_ids': [171, 132, 363, 304, 195, 84],
 'atlas_id': 262,
 'children_structure_ids': [171, 195, 304, 363, 84, 132],
 'color_hex_triplet': u'2FA850',
 'graph_order': 207,
 'hemisphere_id': 3,
 'id': 972,
 'name': u'Prelimbic area',
 'ontology_id': 1,
 'parent_structure_id': 315,
 'st_level': None}
{% endhighlight %}

We can see, among other things, its full name, acronym, and most importantly,
the IDs of its substructures at the lowest level (`annotation_ids`), which
are exactly the values we see in the annotation volume that we imported
in the previous section.

Unfortunately, the Interactive Atlas Viewer currently doesn't provide the 
IDs of selected structures. We can, however, use the following method to
get the ID of a structure given its acronym:

{% highlight python %}
def get_id_by_acronym(structures, acronym):
    structid = [sid for sid in structures
        if structures[sid]['acronym'] == acronym]

    if structid:
        return structid[0]
    else:
        return None
{% endhighlight %}

Accordingly, the output of `get_id_by_acronym(structures, 'PL')` is the value
972.

Creating structure masks
------------------------

The annotation volume extracted above can be saved as a NIfTI file
following the procedure described [last 
time]({% post_url 2015-02-01-working-with-the-allen-mouse-brain-reference-atlas %}#saving-the-volume-in-the-nifti-format).
However, as we said before, this volume contains only annotations of the finest
level structures. More often, we are interested in higher level structures.
For example, in the mouse mesoscale connectome paper, published by 
Allen Institute researchers {% cite Oh:etal:2014 %}, the connectivity
between 295 gray matter structures from the middle level of the 
neuroanatomical ontology was studied. It is therefore important to be 
able to create masks for structures at any level of the hierarchy. And that's
exactly the functionality provided by the following method:

{% highlight python %}
def save_structure_mask(structure_id, filename, structures, annotations):
    values = np.in1d(annotations.ravel(), 
        structures[structure_id]['annotation_ids']).reshape(annots.shape)
    
    h = nib.Nifti1Header()
    h.set_data_shape(values.shape)
    h.set_zooms([0.025, 0.025, 0.025])
    h.set_sform([[0,0,1,0],[-1,0,0,0],[0,-1,0,0]], code='aligned')
    h.set_data_dtype(np.uint8)
    
    f = nib.Nifti1Image(values.astype(np.uint8), None, header=h)
    
    nib.save(f, filename)
{% endhighlight %}

Let's say we need a mask of thalamus:

{% highlight python %}
structures = read_ontology()
annotations = read_annotations()

acronym = 'TH'
filename = '/path/to/output/mask_thalamus.nii.gz'
structure_id = get_id_by_acronym(structures, acronym)

save_structure_mask(structure_id, filename, structures, annotations)
{% endhighlight %}

Masks for all structures in the ontology can be created enclosing the 
last four lines in a simple [`for` 
loop](http://en.wikipedia.org/wiki/For_loop).

And this is it. Again, in case you have any questions, suggestions, etc.,
don't hesitate to contact me.

References
----------

{% bibliography --cited %}
