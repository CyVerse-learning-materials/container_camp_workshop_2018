Introduction to Singularity
---------------------------

|singularity|

1. Prerequisites
================

There are no specific skills needed for this tutorial beyond a basic comfort with the command line and using a text editor. Prior experience in developing web applications will be helpful but is not required.

.. Note:: 
      
      *Important*: Docker and Singularity are `friends <http://singularity.lbl.gov/docs-docker>`_ but they have distinct differences.
      
      **Docker**:
      
      * Docker can escalate privileges, effectively making you root on the host system (This is usually not supported by administrators from High Performance Computing (HPC) centers)
      
      **Singularity**:
     
      * Works on HPC resources
      * Same user inside and outside the container
      * User has root privileges
      * can run (and modify!) existing Docker containers

2. Singularity Installation
===========================

Singularity is more likely to be used in a remote system, e.g. HPC or cloud. Yet you may want to develop containers first on a local machine.

Exercise 1 (~10 mins)
~~~~~~~~~~~~~~~~~~~~~

2.1 Laptop or Desktop
~~~~~~~~~~~~~~~~~~~~~

To Install Singularity on your laptop or desktop PC: (`Mac <http://singularity.lbl.gov/install-mac>`_, `Windows <http://singularity.lbl.gov/install-windows>`_, `Linux <http://singularity.lbl.gov/install-linux>`_)

  * running VM is required on Mac OS X, Singularityware `VagrantBox <https://app.vagrantup.com/singularityware/boxes/singularity-2.4/versions/2.4>`_
  
2.2 HPC
~~~~~~~
Load Singularity module on HPC/XSEDE

If you are working on HPC, you may need to contact your systems administrator and request they install `Singularity  <http://singularity.lbl.gov/install-request>`_. 

Most HPC systems are running Environment Modules with the simple command `module`. You can check to see what is available:

.. code-block:: bash

  $ module avail

If Singularity is installed:

.. code-block:: bash

	$ module load singularity

2.3 XSEDE Jetstream / CyVerse Atmosphere Clouds
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

CyVerse staff have deployed an Ansible playbook with an EZ installation option for `Singularity <https://cyverse-ez-quickstart.readthedocs-hosted.com/en/latest/#>`_ that only requires you to type a short line of code:

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

3. Running Singularity containers from prebuilt images
======================================================

Exercise 2 (~10 mins)
~~~~~~~~~~~~~~~~~~~~~

You can use the `pull` command to download pre-built images from the `Singularity-Hub <https://www.singularity-hub.org>`_ or `DockerHub <https://hub.docker.com/>`_.

This example pulls a container from Docker Hub:

.. code-block:: bash

    $ singularity pull shub://singularityhub/ubuntu
  
You can also rename the container by using the `--name` flag:
  
.. code-block:: bash

    $ singularity pull --name ubuntu_test.simg shub://singularityhub/ubuntu

- Running a Singularity container from pre-built image

After your image has finished downloading it should be in the present working directory, unless you specified to download it somewhere else.

.. code-block:: bash

    $ singularity run ubuntu_test.simg
    
    Singularity ubuntu_test.simg:~> cat /etc/*release
    
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

Downloading images from DockerHub

.. code-block:: bash

	$ singularity pull docker://ubuntu

Keeping track of downloaded images may be necessary if space is a concern. 

By default, Singularity uses a temporary cache to hold Docker tarballs:

.. code-block:: bash

  $ ls ~/.singularity
  
You can change these by specifying the location of the cache and temporary directory:

.. code-block:: bash

  $ sudo mkdir tmp
  $ sudo mkdir scratch
  
  $ SINGULARITY_TMPDIR=$PWD/scratch SINGULARITY_CACHEDIR=$PWD/tmp singularity --debug pull --name ubuntu-tmpdir.simg docker://ubuntu

4. Building Singularity containers locally
==========================================

Like Docker which uses a `dockerfile` to build its containers, Singularity uses a file called `Singularity`

When you are building locally, you can name this file whatever you wish, but a better practice is to put it in a directory and name it `Singularity` - as this will help later on when developing on Singularity-Hub and Github.

.. code-block:: bash

	$ singularity build --name ubuntu.simg Singularity

In the above command:

-	`--name` will create a container named  `ubuntu.simg`

.. Note::

    Bootstrapping `bootstrap` command is deprecated (v2.4), use `build` instead.
    
    To install Ubuntu from the ubuntu.com archive you need to use `debootstrap`

 
Exercise 3: Writing a Singularity file (30 minutes)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

`Recipes <http://singularity.lbl.gov/docs-recipes>`_ for Singularity can use any number of container registries for bootstrapping a container. 

(Advanced) the `Singularity` file can be hosted on Github and will be auto-detected by Singularity-Hub when you set up your Container Collection.

- The Header  

Container Registries: 

* shub - images hosted on Singularity Hub
* docker - images hosted on Docker Hub
* localimage - images saved on your machine
* yum - yum based systems such as CentOS and Scientific Linux
* debootstrap - apt based systems such as Debian and Ubuntu
* arch - Arch Linux
* busybox - BusyBox
* zypper - zypper based systems such as Suse and OpenSuse

- Sections

Singularity files use sections to specify the environment and applications to build.

*  %help - create text for a help menu associated with your container
*  %setup - executed on the host system outside of the container, after the base OS has been installed.
*  %files - copy files from your host system into the container
*  %labels - 
*  %environment - 
*  %post - 
*  %runscript - 
*  %test - 

- Setting up Singularity file system

`$SINGULARITY_ROOTFS`

Example Singularity file using a Docker image of Ubuntu 16.04:

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

.. code-block:: bash

    $ singularity pull  shub://vsoch/hello-world
    Progress |===================================| 100.0% 
    Done. Container is at: /home/tyson_swetnam/vsoch-hello-world-master-latest.simg
    
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

.. code-block:: bash

    $ singularity shell --writable shub://singularityhub/ubuntu

You can convert these images to writable versions using the `--writable` and `--sandbox` commands. 

.. code-block:: bash

    sudo singularity build --sandbox ubuntu/ shub://singularityhub/ubuntu


.. |singularity| image:: ../img/singularity.png
  :width: 300
  :height: 300
