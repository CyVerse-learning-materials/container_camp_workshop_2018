Introduction to Singularity
---------------------------

|singularity|

1. Prerequisites
================

There are no specific skills needed for this tutorial beyond a basic comfort with the command line and using a text editor. Prior experience in developing web applications will be helpful but is not required.

.. Note:: 
      
      *Important*: Docker and Singularity are `friends <http://singularity.lbl.gov/docs-docker>`_ but they have distinct differences. Gregory Kurtzer, creator of Singularity has provided two good talks online: `Introduction to Singularity <https://wilsonweb.fnal.gov/slides/hpc-containers-singularity-introductory.pdf>`_, and `Advanced Singularity <https://www.intel.com/content/dam/www/public/us/en/documents/presentation/hpc-containers-singularity-advanced.pdf>`_. Vanessa Sochat also has given a great talk on `Singularity <https://docs.google.com/presentation/d/14-iKKUpGJC_1qpVFVUyUaitc8xFSw9Rp3v_UE9IGgjM/pub?start=false&loop=false&delayms=3000&slide=id.g1c1cec989b_0_154>`_ which you can see online. 
      
      **Docker**:
      
      * Docker can escalate privileges, effectively making you root on the host system (This is usually not supported by administrators from High Performance Computing (HPC) centers)
      
      **Singularity**:
     
      * Works on HPC resources
      * Same user inside and outside the container
      * User has root privileges
      * can run (and modify!) existing Docker containers

Singularity uses a 'flow' whereby you can (1) create and modify images on your dev system, (2) build containers using recipes or pulling from repositories, and (3) execute containers on production systems. 

|singularityflow|

2. Singularity Installation
===========================

Singularity is more likely to be used in a remote system, e.g. HPC or cloud. Yet you may want to develop containers first on a local machine.

Exercise 1 (15-20 mins)
~~~~~~~~~~~~~~~~~~~~~

2.1 Setting up your Laptop
~~~~~~~~~~~~~~~~~~~~~~~~~~

To Install Singularity on your laptop or desktop PC follow the instructions from Singularity: (`Mac <http://singularity.lbl.gov/install-mac>`_, `Windows <http://singularity.lbl.gov/install-windows>`_, `Linux <http://singularity.lbl.gov/install-linux>`_)

  * running a VM is required on Mac OS X, Singularityware `VagrantBox <https://app.vagrantup.com/singularityware/boxes/singularity-2.4/versions/2.4>`_
  
2.2 HPC
~~~~~~~

Load the Singularity module on a HPC

If you are interested in working on HPC, you may need to contact your systems administrator and request they install `Singularity  <http://singularity.lbl.gov/install-request>`_. 

Most HPC systems are running Environment Modules with the simple command `module`. You can check to see what is available:

.. code-block:: bash

  $ module avail

If Singularity is installed:

.. code-block:: bash

	$ module load singularity

2.3 XSEDE Jetstream / CyVerse Atmosphere Clouds
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

CyVerse staff have deployed an Ansible playbooks called `ez` installation which includes `Singularity <https://cyverse-ez-quickstart.readthedocs-hosted.com/en/latest/#>`_ that only requires you to type a short line of code.

Start a featured instance on Atmosphere or Jetstream.

Type in the following:

.. code-block:: bash

    $ ezs 
    
    * Updating ez singularity and installing singularity (this may take a few minutes, coffee break!)
    Cloning into '/opt/cyverse-ez-singularity'...
    remote: Counting objects: 11, done.
    remote: Total 11 (delta 0), reused 0 (delta 0), pack-reused 11
    Unpacking objects: 100% (11/11), done.
    Checking connectivity... done.

2.4 Check Installation
~~~~~~~~~~~~~~~~~~~~~~

Singularity should now be installed on your laptop or VM, or loaded on the HPC, you can check the installation with:

.. code-block:: bash

    $ singularity pull shub://vsoch/hello-world
    Progress |===================================| 100.0%
    Done. Container is at: /tmp/vsoch-hello-world-master.simg
    $ singularity run vsoch-hello-world-master.simg
    RaawwWWWWWRRRR!!

3. Downloading Singularity containers
=====================================

Exercise 2.1: Downloading from Singularity Hub (~10 mins)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

You can use the `pull` command to download pre-built images from a number of Container Registries, here we'll be focusing on the `Singularity-Hub <https://www.singularity-hub.org>`_ or `DockerHub <https://hub.docker.com/>`_.

Container Registries: 

* shub - images hosted on Singularity Hub
* docker - images hosted on Docker Hub
* localimage - images saved on your machine
* yum - yum based systems such as CentOS and Scientific Linux
* debootstrap - apt based systems such as Debian and Ubuntu
* arch - Arch Linux
* busybox - BusyBox
* zypper - zypper based systems such as Suse and OpenSuse

This example pulls a container from Singularity-Hub:

.. code-block:: bash

    $ singularity pull shub://singularityhub/ubuntu
  
You can also rename the container by using the `--name` flag:
  
.. code-block:: bash

    $ singularity pull --name ubuntu_test.simg shub://singularityhub/ubuntu

- Running a Singularity container from pre-built image

After your image has finished downloading it should be in the present working directory, unless you specified to download it somewhere else.

.. code-block:: bash


	$ singularity pull --name ubuntu_test.simg shub://singularityhub/ubuntu
	Progress |===================================| 100.0% 
	Done. Container is at: /home/***/ubuntu_test.simg
	$ singularity run ubuntu_test.simg 
	This is what happens when you run the container...
	$ singularity shell ubuntu_test.simg 
	Singularity: Invoking an interactive shell within container...

	Singularity ubuntu_test.simg:~> cat /etc/*release
	DISTRIB_ID=Ubuntu
	DISTRIB_RELEASE=14.04
	DISTRIB_CODENAME=trusty
	DISTRIB_DESCRIPTION="Ubuntu 14.04 LTS"
	NAME="Ubuntu"
	VERSION="14.04, Trusty Tahr"
	ID=ubuntu
	ID_LIKE=debian
	PRETTY_NAME="Ubuntu 14.04 LTS"
	VERSION_ID="14.04"
	HOME_URL="http://www.ubuntu.com/"
	SUPPORT_URL="http://help.ubuntu.com/"
	BUG_REPORT_URL="http://bugs.launchpad.net/ubuntu/"
	Singularity ubuntu_test.simg:~> 

Exercise 2.2: Downloading from Docker Hub
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

This example pulls a container from DockerHub

.. code-block:: bash

	$ singularity pull --name ubuntu_docker.simg docker://ubuntu
   	Importing: /home/***/.singularity/docker/sha256:c71a6f8e13782fed125f2247931c3eb20cc0e6428a5d79edb546f1f1405f0e49.tar.gz
	Importing: /home/***/.singularity/docker/sha256:4be3072e5a37392e32f632bb234c0b461ff5675ab7e362afad6359fbd36884af.tar.gz
	Importing: /home/***/.singularity/docker/sha256:06c6d2f5970057aef3aef6da60f0fde280db1c077f0cd88ca33ec1a70a9c7b58.tar.gz
	Importing: /home/***/.singularity/metadata/sha256:c6a9ef4b9995d615851d7786fbc2fe72f72321bee1a87d66919b881a0336525a.tar.gz
	WARNING: Building container as an unprivileged user. If you run this container as root
	WARNING: it may be missing some functionality.
	Building Singularity image...
	Singularity container built: ./ubuntu_docker.simg
	Cleaning up...
	Done. Container is at: ./ubuntu_docker.simg
	
	$ singularity run ubuntu_docker.simg 
	$ cat /etc/*release
	DISTRIB_ID=Ubuntu
	DISTRIB_RELEASE=16.04
	DISTRIB_CODENAME=xenial
	DISTRIB_DESCRIPTION="Ubuntu 16.04.3 LTS"
	NAME="Ubuntu"
	VERSION="16.04.3 LTS (Xenial Xerus)"
	ID=ubuntu
	ID_LIKE=debian
	PRETTY_NAME="Ubuntu 16.04.3 LTS"
	VERSION_ID="16.04"
	HOME_URL="http://www.ubuntu.com/"
	SUPPORT_URL="http://help.ubuntu.com/"
	BUG_REPORT_URL="http://bugs.launchpad.net/ubuntu/"
	VERSION_CODENAME=xenial
	UBUNTU_CODENAME=xenial

Whoa, we're inside the container!?!

This is the OS on the VM I tested this on:

.. code-block:: bash 

	$ exit
	exit
	$ cat /etc/*release
	DISTRIB_ID=Ubuntu
	DISTRIB_RELEASE=16.04
	DISTRIB_CODENAME=xenial
	DISTRIB_DESCRIPTION="Ubuntu 16.04.1 LTS"
	NAME="Ubuntu"
	VERSION="16.04.1 LTS (Xenial Xerus)"
	ID=ubuntu
	ID_LIKE=debian
	PRETTY_NAME="Ubuntu 16.04.1 LTS"
	VERSION_ID="16.04"
	HOME_URL="http://www.ubuntu.com/"
	SUPPORT_URL="http://help.ubuntu.com/"
	BUG_REPORT_URL="http://bugs.launchpad.net/ubuntu/"
	VERSION_CODENAME=xenial
	UBUNTU_CODENAME=xenial

Here we are back in the container:

.. code-block:: bash

	$ singularity shell ubuntu_docker.simg 
	Singularity: Invoking an interactive shell within container...

	Singularity ubuntu_docker.simg:~> cat /etc/*release
	DISTRIB_ID=Ubuntu
	DISTRIB_RELEASE=16.04
	DISTRIB_CODENAME=xenial
	DISTRIB_DESCRIPTION="Ubuntu 16.04.3 LTS"
	NAME="Ubuntu"
	VERSION="16.04.3 LTS (Xenial Xerus)"
	ID=ubuntu
	ID_LIKE=debian
	PRETTY_NAME="Ubuntu 16.04.3 LTS"
	VERSION_ID="16.04"
	HOME_URL="http://www.ubuntu.com/"
	SUPPORT_URL="http://help.ubuntu.com/"
	BUG_REPORT_URL="http://bugs.launchpad.net/ubuntu/"
	VERSION_CODENAME=xenial
	UBUNTU_CODENAME=xenial
	Singularity ubuntu_docker.simg:~> 

Keeping track of downloaded images may be necessary if space is a concern. 

By default, Singularity uses a temporary cache to hold Docker tarballs:

.. code-block:: bash

  $ ls ~/.singularity
  
You can change these by specifying the location of the cache and temporary directory:

.. code-block:: bash

  $ sudo mkdir tmp
  $ sudo mkdir scratch
  
  $ SINGULARITY_TMPDIR=$PWD/scratch SINGULARITY_CACHEDIR=$PWD/tmp singularity --debug pull --name ubuntu-tmpdir.simg docker://ubuntu

We can also run a docker container in Singularity that launches a program, for example RStudio `tidyverse` from `Rocker <https://hub.docker.com/r/rocker/rstudio/>`_ 

.. code-block:: bash

	$ singularity exec docker://rocker/tidyverse:latest R

`"An Introduction to Rocker: Docker Containers for R by Carl Boettiger, Dirk Eddelbuettel" <https://journal.r-project.org/archive/2017/RJ-2017-065/RJ-2017-065.pdf>`_ 

4. Building Singularity containers locally
==========================================

Like Docker which uses a `dockerfile` to build its containers, Singularity uses a file called `Singularity`

When you are building locally, you can name this file whatever you wish, but a better practice is to put it in a directory and name it `Singularity` - as this will help later on when developing on Singularity-Hub and Github.

Create Container and add content to it:

.. code-block:: bash

	$ singularity image.create ubuntu14.simg
	Creating empty 768MiB image file: ubuntu14.simg
	Formatting image with ext3 file system
	Image is done: ubuntu14.simg

	$ singularity build ubuntu14.simg docker://ubuntu:14.04
	Building into existing container: ubuntu14.simg
	Docker image path: index.docker.io/library/ubuntu:14.04
	Cache folder set to /home/.../.singularity/docker
	[5/5] |===================================| 100.0% 
	Importing: base Singularity environment
	Importing: /home/.../.singularity/docker/sha256:c954d15f947c57e059f67a156ff2e4c36f4f3e59b37467ff865214a88ebc54d6.tar.gz
	Importing: /home/.../.singularity/docker/sha256:c3688624ef2b94ab3981564e23e1f48df8f1b988519373ccfb79d7974017cb85.tar.gz
	Importing: /home/.../.singularity/docker/sha256:848fe4263b3b44987f0eacdb2fc0469ae6ff04b2311e759985dfd27ae5d3641d.tar.gz
	Importing: /home/.../.singularity/docker/sha256:23b4459d3b04aa0bc7cb7f7021e4d7bbb5e87aa74a6a5f57475a0e8badbd9a26.tar.gz
	Importing: /home/.../.singularity/docker/sha256:36ab3b56c8f1a3188464886cbe41f42a969e6f9374e040f13803d796ed27b0ec.tar.gz
	Importing: /home/.../.singularity/metadata/sha256:c6a9ef4b9995d615851d7786fbc2fe72f72321bee1a87d66919b881a0336525a.tar.gz
	WARNING: Building container as an unprivileged user. If you run this container as root
	WARNING: it may be missing some functionality.
	Building Singularity image...
	Singularity container built: ubuntu14.simg
	Cleaning up...

Note, `image.create` uses an ext3 file system

Build to your container by pulling an image from Docker:

	$ singularity pull docker://ubuntu:16.04
	WARNING: pull for Docker Hub is not guaranteed to produce the
	WARNING: same image on repeated pull. Use Singularity Registry
	WARNING: (shub://) to pull exactly equivalent images.
	Docker image path: index.docker.io/library/ubuntu:16.04
	Cache folder set to /home/.../.singularity/docker
	[5/5] |===================================| 100.0% 
	Importing: base Singularity environment
	Importing: /home/.../.singularity/docker/sha256:1be7f2b886e89a58e59c4e685fcc5905a26ddef3201f290b96f1eff7d778e122.tar.gz
	Importing: /home/.../.singularity/docker/sha256:6fbc4a21b806838b63b774b338c6ad19d696a9e655f50b4e358cc4006c3baa79.tar.gz
	Importing: /home/.../.singularity/docker/sha256:c71a6f8e13782fed125f2247931c3eb20cc0e6428a5d79edb546f1f1405f0e49.tar.gz
	Importing: /home/.../.singularity/docker/sha256:4be3072e5a37392e32f632bb234c0b461ff5675ab7e362afad6359fbd36884af.tar.gz
	Importing: /home/.../.singularity/docker/sha256:06c6d2f5970057aef3aef6da60f0fde280db1c077f0cd88ca33ec1a70a9c7b58.tar.gz
	Importing: /home/.../.singularity/metadata/sha256:c6a9ef4b9995d615851d7786fbc2fe72f72321bee1a87d66919b881a0336525a.tar.gz
	WARNING: Building container as an unprivileged user. If you run this container as root
	WARNING: it may be missing some functionality.
	Building Singularity image...
	Singularity container built: ./ubuntu-16.04.simg
	Cleaning up...
	Done. Container is at: ./ubuntu-16.04.simg
	
Create a container using a custom Singularity file:

.. code-block:: bash

	$ singularity build --name ubuntu.simg Singularity

In the above command:

-	`--name` will create a container named  `ubuntu.simg`

Create a writable container using the `--writable` flag:

.. code-block:: bash
	

.. Note::

    Bootstrapping `bootstrap` command is deprecated (v2.4), use `build` instead.
    
    To install a container with Ubuntu from the ubuntu.com reposutiry you need to use `debootstrap`

 
Exercise 3: Creating the Singularity file (30 minutes)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

`Recipes <http://singularity.lbl.gov/docs-recipes>`_ can use any number of container registries for bootstrapping a container. 

(Advanced) the `Singularity` file can be hosted on Github and will be auto-detected by Singularity-Hub when you set up your Container Collection.

- The Header  

The container registries

- Sections

The Singularity file uses sections to specify the dependencies, environmental settings, and runscripts when it build.

*  %help - create text for a help menu associated with your container
*  %setup - executed on the host system outside of the container, after the base OS has been installed.
*  %files - copy files from your host system into the container
*  %labels - 
*  %environment - exports environment settings to the container
*  %post - use to install software and dependencies
*  %runscript - executes a script when the container runs
*  %test - runs a test on the build of the container

- Setting up Singularity file system

`$SINGULARITY_ROOTFS`

Example Singularity file using a `Docker hosted version <https://hub.docker.com/_/ubuntu/>`_ of Ubuntu (16.04 here):

.. code-block:: bash

    BootStrap: docker
    From: ubuntu:16.04

    %post
        apt-get -y update
        apt-get -y install fortune cowsay lolcat

    %environment
        export LC_ALL=C
        export PATH=/usr/games:$PATH

    %runscript
        fortune | cowsay | lolcat
    
Build the container:

.. code-block:: bash

    singularity build --name cowsay_container.simg Singularity

Run the container:

.. code-block:: bash

    singularity run cowsay.simg

If you build a `squashfs` container, it is immutable (you cannot `--writable` edit it)

5. Running Singularity Containers
=================================

Commands:

`exec` - command allows you to execute a custom command within a container by specifying the image file.

`shell` - command allows you to spawn a new shell within your container and interact with it.

`run` - assumes your container is set up with "runscripts" triggered with the `run` command, or simply by calling the container as though it were an executable.

`inspect` - inspects the container.

`--writable` - creates a writable container that you can edit interactively and save on exit.

5.1 Using the `exec` command
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: bash

    $ singularity exec shub://singularityhub/ubuntu cat /etc/os-release


5.2 Using the `shell` command
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: bash

    $ singularity shell shub://singularityhub/ubuntu


5.3 Using the `run` command
~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: bash

    $ singularity run shub://singularityhub/ubuntu
    

5.4 Using the `inspect` command
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

You can inspect the build of your container using the `inspect` command

.. code-block:: bash

    $ singularity pull  shub://vsoch/hello-world
    Progress |===================================| 100.0% 
    Done. Container is at: /home/***/vsoch-hello-world-master-latest.simg
    
    $ singularity inspect vsoch-hello-world-master-latest.simg 
    {
        "org.label-schema.usage.singularity.deffile.bootstrap": "docker",
        "MAINTAINER": "vanessasaur",
        "org.label-schema.usage.singularity.deffile": "Singularity",
        "org.label-schema.schema-version": "1.0",
        "WHATAMI": "dinosaur",
        "org.label-schema.usage.singularity.deffile.from": "ubuntu:14.04",
        "org.label-schema.build-date": "2017-10-15T12:52:56+00:00",
        "org.label-schema.usage.singularity.version": "2.4-feature-squashbuild-secbuild.g780c84d",
        "org.label-schema.build-size": "333MB"
    }

5.4 Using the `--sandbox` and `--writable` commands
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

As of Singularity v2.4 by default `build` produces immutable images in the 'squashfs' file format. This ensures reproducible and verifiable images.

Creating a `--writable` image must be done using the `sudo` command, thus the owner of the container is `root`

.. code-block:: bash

   	$ sudo singularity build --writable ubuntu-master.simg shub://singularityhub/ubuntu
	Cache folder set to /root/.singularity/shub
	Progress |===================================| 100.0% 
	Building from local image: /root/.singularity/shub/singularityhub-ubuntu-master-latest.simg
	Creating empty Singularity writable container 208MB
	Creating empty 260MiB image file: ubuntu-master.simg
	Formatting image with ext3 file system
	Image is done: ubuntu-master.simg
	Building Singularity image...
	Singularity container built: ubuntu-master.simg
	Cleaning up...

You can convert these images to writable versions using the `--writable` and `--sandbox` commands. 

When you use the `--sandbox` the container is written into a directory structure. Sandbox folders can be created without the `sudo` command.

.. code-block:: bash

    	$ singularity build --sandbox lolcow/ shub://GodloveD/lolcow
	WARNING: Building sandbox as non-root may result in wrong file permissions
	Cache folder set to /home/.../.singularity/shub
	Progress |===================================| 100.0% 
	Building from local image: /home/.../.singularity/shub/GodloveD-lolcow-master-latest.simg
	WARNING: Building container as an unprivileged user. If you run this container as root
	WARNING: it may be missing some functionality.
	Singularity container built: lolcow/
	Cleaning up...
	@vm142-73:~$ cd lolcow/
	@vm142-73:~/lolcow$ ls
	bin  boot  dev  environment  etc  home  lib  lib64  media  mnt  opt  proc  run  sbin  singularity  srv  sys  tmp  usr  var

.. |singularity| image:: ../img/singularity.png

.. |singularityflow| image:: http://singularity.lbl.gov/assets/img/diagram/singularity-2.4-flow.png
  :width: 300
  :height: 300
