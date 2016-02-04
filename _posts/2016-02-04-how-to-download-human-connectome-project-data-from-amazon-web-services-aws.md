---
layout: post
title: How to download Human Connectome Project data from Amazon Web Services
---

All public releases of data produced by the 
[Human Connectome Project](http://www.humanconnectome.org/) (HCP) are 
available online through the [ConnectomeDB](https://db.humanconnectome.org).
Nevertheless, I've had trouble running the Aspera Connect browser plugin 
-- that's required to download data from the database -- on my Ubuntu machine
(it crashes whenever I want to initiate a download).

Fortunately, HCP has decided to start sharing all the data also through
Amazon Web Services (AWS), as part of the 
[AWS Public Data Sets](https://aws.amazon.com/public-data-sets/) program,
which makes the data available directly from the command line using
[AWS Command Line Interface](https://aws.amazon.com/cli/) (`awscli`).

In order to access the data using `awscli`, you first need to create
your AWS credentials through the ConnectomeDB (the steps are described
[here](https://wiki.humanconnectome.org/display/PublicData/How+To+Connect+to+Connectome+Data+via+AWS))
and then setup your `awscli` accordingly (see the documentation 
[here](http://docs.aws.amazon.com/cli/latest/userguide/cli-chap-welcome.html)).

In case you're not familiar with the HCP directory structure, 
it will be very useful to read about it before trying to access and 
download the data. The directory structure
is described in the reference manuals that come with every release 
and these are accessible on the HCP site under
[Documentation](http://humanconnectome.org/documentation/).

I usually create simple Bash scripts to fetch all the data that I need. 
The following code, for example, downloads the preprocessed diffusion 
data for 10 unrelated subjects ([text file](/downloads/code/unrelated_10.txt)
 with the list of subjects):

{% highlight bash %}
#!/bin/bash

subjectlist=unrelated_10.txt

while read -r subject;
do
    mkdir -p hcp/$subject
    mkdir -p hcp/$subject/T1w
    mkdir -p hcp/$subject/T1w/Diffusion

    aws s3 cp \
        s3://hcp-openaccess/HCP/$subject/T1w/T1w_acpc_dc_restore_1.25.nii.gz \
        hcp/$subject/T1w \
        --region us-east-1

    aws s3 cp \
        s3://hcp-openaccess/HCP/$subject/T1w/Diffusion \
        hcp/$subject/T1w/Diffusion \
        --recursive \
        --region us-east-1

done < $subjectlist
{% endhighlight %} 
