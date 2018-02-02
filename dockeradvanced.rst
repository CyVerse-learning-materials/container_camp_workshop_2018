Advanced docker
---------------

Now that we are relatively comfortable with Docker basics, lets look at some of the advanced Docker topics such as porting the Docker image to repositories (public and private), managing data in containers and finally deploy containers into cloud and other infrastructures etc.,

Docker registries
=================

To demonstrate the portability of what we just created, let’s upload our built Docker image and run it somewhere else (Atmosphere cloud). After all, you’ll need to learn how to push to registries when you want to deploy containers to production.

.. important::

	So what exact is a registry?

	A registry is a collection of repositories, and a repository is a collection of images—sort of like a GitHub repository, except the code is already built. An account on a registry can create many repositories. The docker CLI uses Docker’s public registry by default. You can even set up your own private registry using Docker Trusted Registry

There are several things you can do with Docker registries:

- Pushing images 
- Finding images
- Pulling images
- Sharing images

Public repositories 
~~~~~~~~~~~~~~~~~~~

Some example of public registries include `Docker cloud <https://cloud.docker.com/>`_, `Docker hub <https://hub.docker.com/>`_ and `quay.io <https://quay.io/>`_)

Log in with your Docker ID
^^^^^^^^^^^^^^^^^^^^^^^^^^

Now that you've created and tested your image, you can push it to Docker cloud or Docker hub.

.. Note::

	If you don’t have a Docker account, sign up for one at Docker cloud `Docker cloud <https://cloud.docker.com/>`_, `Docker hub <https://hub.docker.com/>`. Make note of your username. There are several advantages of registering to Dockerhub which we will see later on in the session

First you have to login to your Docker hub account, to do that:

.. code-block:: bash

	$ docker login
	Login with your Docker ID to push and pull images from Docker Hub. If you don't have a Docker ID, head over to https://hub.docker.com to create one.
	Username (upendradevisetty):                 
	Password:

Enter YOUR_USERNAME and password when prompted.

Tag the image
^^^^^^^^^^^^^^

The notation for associating a local image with a repository on a registry is `username/repository:tag`. The tag is optional, but recommended, since it is the mechanism that registries use to give Docker images a version. Give the repository and tag meaningful names for the context, such as `get-started:part2`. This will put the image in the `get-started` repository and tag it as `part2`.

.. Note::

	By default the docker image gets a `latest` tag if you don't provide one. Thought convenient, it is not recommended for reproducibility purposes.

Now, put it all together to tag the image. Run docker tag image with your username, repository, and tag names so that the image will upload to your desired destination. For our docker image since we already have our Dockerhub username we will just add tag which in this case is `1.0`

.. code-block:: bash

	$ docker tag <YOUR_DOCKERHUB_USERNAME>/myfirstapp <YOUR_DOCKERHUB_USERNAME>/myfirstapp:1.0

Publish the image
^^^^^^^^^^^^^^^^^

Upload your tagged image to the Docker repository

.. code-block:: bash

	docker push <YOUR_DOCKERHUB_USERNAME>/myfirstapp:1.0	

Once complete, the results of this upload are publicly available. If you log in to Docker Hub, you will see the new image there, with its pull command.

|docker_image|

Pull and run the image from the remote repository
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

From now on, you can use `docker run` and run your app on any machine with this command

Let's try this on cloud by logging into CyVerse Atmosphere, launching an instance and then running the following command

.. code-block:: bash

	$ docker run -d -p 8888:5000 --name myfirstapp <YOUR_DOCKERHUB_USERNAME>/myfirstapp

You don't have to run `docker pull` since if the image isn’t available locally on the machine, Docker will pull it from the repository.

Private repositories
~~~~~~~~~~~~~~~~~~~~

For this step, we'll need to launch a registry

.. code-block:: bash

	$ docker run -d -p 5000:5000 --name registry registry:2

Run `docker ps -l` to check the recent container from this Docker image

.. code-block:: bash

	$ docker ps -l
	CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                    NAMES
	6e44a0459373        registry:2          "/entrypoint.sh /e..."   11 seconds ago      Up 10 seconds       0.0.0.0:5000->5000/tcp   registry

Tag the image
^^^^^^^^^^^^^^

Then tag your image under the registry namespace and push it there

.. code-block:: bash

	$ REGISTRY=localhost:5000
	$ docker tag myfirstapp $REGISTRY/$(whoami)/myfirstapp:1.0

Publish the image
^^^^^^^^^^^^^^^^^

Now push the image to the local registry

.. code-block:: bash

	$ docker push $REGISTRY/$(whoami)/myfirstapp:1.0
	The push refers to a repository [localhost:5000/upendra_35/myfirstapp]
	64436820c85c: Pushed 
	831cff83ec9e: Pushed 
	c3497b2669a8: Pushed 
	1c5b16094682: Pushed 
	c52044a91867: Pushed 
	60ab55d3379d: Pushed 
	1.0: digest: sha256:5095dea8b2cf308c5866ef646a0e84d494a00ff0e9b2c8e8313a176424a230ce size: 1572

Pull and run the image from the local repository
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Now pull the image from the local repository and run a container from it

.. code-block:: bash

	docker run -d -P --name=myfirstapplocal $REGISTRY/$(whoami)/myfirstapp:1.0

Exercise 1 (10 mins)
~~~~~~~~~~~~~~~~~~~~

- Add more cat pics to the container
- Run an instance to make sure the new pics show up
- Change the tag from 1.0 to 2.0
- Push it to your Dockerhub
- Share your Dockerhub link url on Slack

Solution
~~~~~~~~

Automated Docker image building from github and bitbucket
=========================================================




Exposing ports on running Containers
====================================


Working with volumes (creating and binding volumes)
===================================================


Managing data for analysis in Docker containers
===============================================


Optimizing Dockerfiles: Multi-stage builds
==========================================


Manage sensitive data with Docker secrets
=========================================


Docker Compose for multi container apps
=======================================


Improving your data science workflow using Docker containers
============================================================


Deploy your app
===============