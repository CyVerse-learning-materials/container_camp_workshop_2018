**Distributed Computing with Makeflow and Work Queue**
-------------------------------------------------

1. Prerequisites
================

ssh will be used to connect to a remote job submit host. Please ensure you have a ssh client installed. The instructors will supply a slip of paper with username, password and hostname during the session.

2. What are Scientific Workflows?
=================================

Scientific workflows allow users to easily express multi-step computational tasks, for example retrieve data from an instrument or a database, reformat the data, and run an analysis. A scientific workflow describes the dependencies between the tasks and in most cases the workflow is described as a directed acyclic graph (DAG), where the nodes are tasks and the edges denote the task dependencies. A defining property for a scientific workflow is that it manages data flow. The tasks in a scientific workflow can be everything  from short serial tasks to very large parallel tasks (MPI for example) surrounded by a large number of small, serial tasks used for pre- and post-processing.

|makeflow_dag|

2. Cooperative Computing Lab
============================

The CCL designs software that enables our collaborators to easily harness large scale distributed systems such as clusters, clouds, and grids. We perform fundamental computer science research that enables new discoveries through computing in fields such as physics, chemistry, bioinformatics, biometrics, and data mining.

The software suite we write and maintain is the CCTools software package.

- **`Makeflow<http://ccl.cse.nd.edu/software/makeflow/>`_**. A portable workflow manager.

- **`Work Queue**. A lightweight distributed execution system.

- **`Parrot**. A personal user-level virtual file system.

- **`Chirp**. A user-level distributed filesystem.

- **`Specialized Software**. A selection of applications tailored to specific compuation tasks.

2. Makeflow
===========

Makeflow is a workflow system for executing large complex workflows on clusters, clouds, and grids.

- **Makeflow is easy to use.** The Makeflow language is similar to traditional Make, so if you can write a Makefile, then you can write a Makeflow. A workflow can be just a few commands chained together, or it can be a complex application consisting of thousands of tasks. It can have an arbitrary DAG structure and is not limited to specific patterns.

- **Makeflow is production-ready.** Makeflow is used on a daily basis to execute complex scientific applications in fields such as data mining, high energy physics, image processing, and bioinformatics. It has run on campus clusters, the Open Science Grid, NSF XSEDE machines, NCSA Blue Waters, and Amazon Web Services. Here are some real examples of workflows used in production systems:

- **Makeflow is portable.** A workflow is written in a technology neutral way, and then can be deployed to a variety of different systems without modification, including local execution on a single multicore machine, public cloud services such as Amazon EC2 and Amazon Lambda, batch systems like HTCondor, SGE, PBS, Torque, SLURM, or the bundled Work Queue system. Makeflow can also easily run your jobs in a container environment like Docker or Singularity on top of an existing batch system. The same specification works for all systems, so you can easily move your application from one system to another without rewriting everything.

- **Makeflow is powerful.** Makeflow can handle workloads of millions of jobs running on thousands of machines for months at a time. Makeflow is highly fault tolerant: it can crash or be killed, and upon resuming, will reconnect to running jobs and continue where it left off. A variety of analysis tools are available to understand the performance of your jobs, measure the progress of a workflow, and visualize what is going on.

2. Work Queue
=============

Work Queue is a framework for building large master-worker applications that span thousands of machines drawn from clusters, clouds, and grids. Work Queue applications are written in C, Perl, or Python using a simple API that allows users to define tasks, submit them to the queue, and wait for completion. Tasks are executed by a standard worker process that can run on any available machine. Each worker calls home to the master process, arranges for data transfer, and executes the tasks. The system handles a wide variety of failures, allowing for dynamically scalable and robust applications.

Work Queue has been used to write applications that scale from a handful of workstations up to tens of thousands of cores running on supercomputers. Examples include Lobster, NanoReactors, ForceBalance, Accelerated Weighted Ensemble, the SAND genome assembler, the Makeflow workflow engine, and the All-Pairs and Wavefront abstractions. The framework is easy to use, and has been used to teach courses in parallel computing, cloud computing, distributed computing, and cyberinfrastructure at the University of Notre Dame, the University of Arizona, and the University of Wisconsin - Eau Claire.

3. Makeflow Tutorial
====================

This tutorial goes through the installation process of CCTools, 
the creation and running of a Makeflow, and 
how to use Makeflow in conjunction with Work Queue to leverage different execution resources for your execution. 
More information can be found a `http://ccl.cse.nd.edu/<http://ccl.cse.nd.edu/>`_. For specific information on 
Makeflow execution see `http://ccl.cse.nd.edu/software/manuals/makeflow.html<http://ccl.cse.nd.edu/software/manuals/makeflow.html>`_ and 
Work Queue see `http://ccl.cse.nd.edu/software/manuals/workqueue.html<http://ccl.cse.nd.edu/software/manuals/workqueue.html>`_.

3.1. Running on Atmosphere/Jetstream
====================================

To start out we are going to launch an instance:





3.2. Download and Installation
==============================

If you have access to the Notre Dame Center for Research Computing, first log into the CRC head node <tt>crcfe01.crc.nd.edu</tt> by using <tt>ssh</tt>, PuTTY, or a similar tool. If you do not have access, please build the code on your own machine. Once you have a shell, download and install the CCTools software in your home directory in one of two ways:
<p>

To build our latest release:

.. code-block:: bash

    $ wget http://ccl.cse.nd.edu/software/files/cctools-6.2.6-source.tar.gz
    $ tar zxpvf cctools-6.2.6-source.tar.gz
    $ cd cctools-6.2.6-source
    $ ./configure --prefix $HOME/cctools --tcp-low-port 9000 --tcp-high-port 9500
    $ make
    $ make install
    $ cd $HOME


If you use bash then do this to set your path:

.. code-block:: bash

    $ export PATH=$HOME/cctools/bin:$PATH

If you use tcsh instead, then do this:

.. code-block:: bash

    $ setenv PATH $HOME/cctools/bin:$PATH

Now double check that you can run the various commands, like this:

.. code-block:: bash

    $ makeflow -v
    $ work_queue_worker -v
    $ work_queue_status

3.3. Getting Makeflow-Examples
==============================


3. Makeflow Example
===================

Let's begin by using Makeflow to run a handful of simulation codes.
First, make and enter a clean directory to work in:

.. code-block:: bash

    $ cd $HOME
    $ mkdir tutorial
    $ cd tutorial

Download this program, which performs a highly sophisticated simulation of black holes colliding together:

.. code-block:: bash

    $ wget http://ccl.cse.nd.edu/software/tutorials/cyversecc18/simulation.py

Try running it once, just to see what it does:

.. code-block:: bash

    $ chmod 755 simulation.py
    $ ./simulation.py 5

Now, let's use Makeflow to run several simulations.
Create a file called <tt>example.makeflow</tt> and paste the following
text into it:

.. code-block:: text

    input.txt:
    	LOCAL /bin/echo "Hello Makeflow!" > input.txt

    output.1: simulation.py input.txt
    	./simulation.py 1 < input.txt > output.1

    output.2: simulation.py input.txt
    	./simulation.py 2 < input.txt > output.2

    output.3: simulation.py input.txt
    	./simulation.py 3 < input.txt > output.3

    output.4: simulation.py input.txt
    	./simulation.py 4 < input.txt > output.4

To run it on your local machine, one job at a time:

.. code-block:: bash

    $ makeflow example.makeflow -j 1

Note that if you run it a second time, nothing will happen, because all of the files are built:

.. code-block:: bash

    $ makeflow example.makeflow
    $ makeflow: nothing left to do

Use the -c option to clean everything up before trying it again:

.. code-block:: bash

    $ makeflow -c example.makeflow

Here are some other options for built-in batch systems:

.. code-block:: bash

    $ makeflow -T slurm example.makeflow
    $ makeflow -T torque example.makeflow
    $ makeflow -T sge example.makeflow

3. Running Makeflow with Work Queue
===================================

You will notice that a workflow can run very slowly if you submit each job individually. To get around this limitation, we provide the Work Queue system. This allows Makeflow to function as a master process that quickly dispatches work to remote worker processes. 

.. code-block:: bash

    $ makeflow -c example.makeflow
    $ makeflow -T wq example.makeflow -p 0
    listening for workers on port <font color=white>XXXX</font>.
    ...

Now open up another shell and run a single worker process:

.. code-block:: bash

    $ work_queue_worker crcfe01.crc.nd.edu <font color=white>XXXX</font>

Go back to your first shell and observe that the makeflow has finished.
Of course, remembering port numbers all the time gets old fast,
so try the same thing again, but using a project name:

.. code-block:: bash

    $ makeflow -c example.makeflow
    $ makeflow -T wq example.makeflow <font color=white>-N project-$USER</font>
    listening for workers on port XXXX
    ...

Now open up another shell and run your worker with a project name:
.. code-block:: bash

    $ work_queue_worker <font color=white>-N project-$USER</font>

4. Work Queue Exercise
======================

