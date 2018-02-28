OSG (Open Science Grid) Singularity Infrastructure
--------------------------------------------------

1. Prerequisites
================

ssh will be used to connect to a remote job submit host. Please ensure you have a ssh client installed. The instructors will supply a slip of paper with username, password and hostname during the session.

2. OSG Overview
===============

The `Open Science Grid (OSG) <https://www.opensciencegrid.org/>`_ is a distributed infrastructure for high throughput computing. The OSG enables dozens of universities and labs to provide researchers with access to significant computing resources. The resources accessible through the OSG are contributed by the community, organized by the OSG, and governed by the OSG consortium. In the last 12 months, we have provided more than 1.2 Billion CPU hours to researchers across a wide variety of projects.

|osg_map|

This map is available "live" on the `OSG Display <https://display.grid.iu.edu/>`_ site.

2.1 Distributed High Throughput Computing
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

OSG is focusing on single core, or low number core jobs which can fit on a single node. Some guidelines:

- Use less than 2 GB memory 

- Each invocation should run for 1-12 hours

- Compute sites in the OSG can be configured to use preemption, which means jobs can be automatically killed if higher priority jobs enter the system. Preempted jobs will restart on another site, but it is important that the jobs can handle multiple restarts.

2.2 Motivations for Containers in OSG
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Each computing resource exposes a slightly different operating system environment. Actually this capacity can be daunting for the individual: each site has its own discretion to setup their runtime environment in a unique way. 

.. Note:: 

	Before containers, OSG **used** to tell the users:
	
	- Binaries should preferably be statically linked. However, dynamically linked binaries with standard library dependencies, built for a 64-bit Red Hat Enterprise Linux (RHEL) 6 machines will also work. Also, interpreted languages such as Python or Perl will work as long as there are no special module requirements.

	- Software dependencies can be difficult to accommodate unless the software can be staged with the job.

Motivation for containers in OSG:

- Consistent environment (default images)

- Custom software environment (user defined images)

- Enabling special environment such as GPUs

- Process isolation

- File isolation

2.3 Container Statistics
~~~~~~~~~~~~~~~~~~~~~~~~

These are for a subset of OSG, specifically for the OSG VO (Virtual Organization). Other VOs are also using containers.

|osg_container_count|

One challenge when running these many container per day, across 100's of sites and 1000's of compute nodes, is how do we distribute and access containers without putting unnecessary load on Docker and Singularity hubs? More about this below.

The breakdown of jobs shows about half runs without containers, and the once running in containers are mostly doing so under the default images.

|osg_container_breakdown|


3. CVMFS
========

The CernVM File System (CVMFS) is a highly-scalable global filesystem optimized for global distribution of software.  The CERN-based LHC experiments invested in this filesystem based on the experience of attempting to synchronize the install their complex application software stacks across hundreds of sites.  Each release may contain tens of gigabytes of data across hundreds of thousands of files; a few dozen to a hundred releases might be active at any given time. 

CVMFS is FUSE-based - a filesystem implemented in user space, not within the Linux kernel.  It scales well because changes to each repository are only written to a single repository node and then distributed throughout the CVMFS content distribution network (a hierarchical set of web servers and HTTP caches).  All writes are aggregated into a single transaction, making the rate of change relatively slow (typically, updates occur no faster than once every 15 minutes).  Since file contents are immutable, CVMFS is able to use a content-addressed scheme and the corresponding HTTP objects immutable.  Thus, the entire system is amenable to cache hierarchies.

CVMFS's original use case has significant parallels with distributing scientific containers: containers tend to be read-only, contain relatively large sets of software, and need to be accessed - without modification or corruption - at multiple sites.

|osg_cvmfs|

OSG stores container images on CVMFS in extracted form. That is, we take the Docker image layers or the Singularity img/simg files and export them onto CVMFS. For example, `ls` on one of the containers looks similar to `ls /` on any Linux machine:

.. code-block:: bash

	$ ls /cvmfs/singularity.opensciencegrid.org/opensciencegrid/osgvo-el7:latest/
	cvmfs  host-libs  proc  sys  anaconda-post.log     lib64
	dev    media      root  tmp  bin                   sbin
	etc    mnt        run   usr  image-build-info.txt  singularity
	home   opt        srv   var  lib

This is a very efficient way for use to distribute the images. Most jobs only need small parts of the actual image (as low as 25-100 MBs), and the CVMFS caching mechanism means those bits are aggressivly cached at both the site and node level.

3.1 cvmfs-singularity-sync
~~~~~~~~~~~~~~~~~~~~~~~~~~


4. Exercise 1: Exploring Available Images
=========================================


5. Exercise 2: Containerized Job - Default Image
================================================


6. Exercise 3: Containerized Job - Custom Image
===============================================



.. |osg_map| image:: ../img/osg_map.png
  :width: 750
  :height: 700 

.. |osg_container_count| image:: ../img/osg_container_count.png
  :width: 750
  :height: 700 

.. |osg_container_breakdown| image:: ../img/osg_container_breakdown.png
  :width: 750
  :height: 700 

.. |osg_cvmfs| image:: ../img/osg_cvmfs.png
  :width: 750
  :height: 700 

