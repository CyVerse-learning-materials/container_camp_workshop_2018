Advanced docker
---------------

Now that we are relatively comfortable with Docker basics, lets look at some of the advanced Docker topics such as porting the Docker image to repositories (public and private), managing data in containers and finally deploy containers into cloud and other infrastructures etc.,

1. Docker registries
====================

To demonstrate the portability of what we just created, let’s upload our built Docker image and run it somewhere else (Atmosphere cloud). After all, you’ll need to learn how to push to registries when you want to deploy containers to production.

.. important::

	So what exact is a registry?

	A registry is a collection of repositories, and a repository is a collection of images—sort of like a GitHub repository, except the code is already built. An account on a registry can create many repositories. The docker CLI uses Docker’s public registry by default. You can even set up your own private registry using Docker Trusted Registry

There are several things you can do with Docker registries:

- Pushing images 
- Finding images
- Pulling images
- Sharing images

1.1 Public repositories 
~~~~~~~~~~~~~~~~~~~~~~~

Some example of public registries include `Docker cloud <https://cloud.docker.com/>`_, `Docker hub <https://hub.docker.com/>`_ and `quay.io <https://quay.io/>`_)

1.1.1 Log in with your Docker ID
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Now that you've created and tested your image, you can push it to Docker cloud or Docker hub.

.. Note::

	If you don’t have a Docker account, sign up for one at `Docker cloud <https://cloud.docker.com/>`_, `Docker hub <https://hub.docker.com/>`_. Make note of your username. There are several advantages of registering to Dockerhub which we will see later on in the session

First you have to login to your Docker hub account, to do that:

.. code-block:: bash

	$ docker login
	Login with your Docker ID to push and pull images from Docker Hub. If you don't have a Docker ID, head over to https://hub.docker.com to create one.
	Username (upendradevisetty):                 
	Password:

Enter Username and Password when prompted.

1.1.2 Tag the image
^^^^^^^^^^^^^^^^^^^

The notation for associating a local image with a repository on a registry is `username/repository:tag`. The tag is optional, but recommended, since it is the mechanism that registries use to give Docker images a version. Give the repository and tag meaningful names for the context, such as `get-started:part2`. This will put the image in the `get-started` repository and tag it as `part2`.

.. Note::

	By default the docker image gets a `latest` tag if you don't provide one. Thought convenient, it is not recommended for reproducibility purposes.

Now, put it all together to tag the image. Run docker tag image with your username, repository, and tag names so that the image will upload to your desired destination. For our docker image since we already have our Dockerhub username we will just add tag which in this case is `1.0`

.. code-block:: bash

	$ docker tag <YOUR_DOCKERHUB_USERNAME>/myfirstapp <YOUR_DOCKERHUB_USERNAME>/myfirstapp:1.0

1.1.3 Publish the image
^^^^^^^^^^^^^^^^^^^^^^^

Upload your tagged image to the Docker repository

.. code-block:: bash

	docker push <YOUR_DOCKERHUB_USERNAME>/myfirstapp:1.0	

Once complete, the results of this upload are publicly available. If you log in to Docker Hub, you will see the new image there, with its pull command.

|docker_image|

1.1.4 Pull and run the image from the remote repository
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Let's try to run the image from the remote repository on cloud by logging into CyVerse Atmosphere, `launching an instance <>`_

First install Docker on Atmosphere using from here https://docs.docker.com/install/linux/docker-ce/ubuntu or alternatively you can use `ez` command which is a short cut command for installing Docker on Atmosphere

.. code-block:: bash

	$ ezd

Now run the following command to pull the docker image from Dockerhub

.. code-block:: bash

	$ sudo docker run -d -p 8888:5000 --name myfirstapp <YOUR_DOCKERHUB_USERNAME>/myfirstapp:1.0

.. Note::

	You don't have to run `docker pull` since if the image isn’t available locally on the machine, Docker will pull it from the repository.

Head over to http://<ipaddress>:8888 and your app should be live. 

1.2 Private repositories
~~~~~~~~~~~~~~~~~~~~~~~~

In an earlier part, we had looked at the Docker Hub, which is a public registry that is hosted by Docker. While the Docker Hub plays an important role in giving public visibility to your Docker images and for you to utilize quality Docker images put up by others, there is a clear need to setup your own private registry too for your team/organization

1.2.1 Pull down the Registry Image
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

You might have guessed by now that the registry must be available as a Docker image from the Docker Hub and it should be as simple as pulling the image down and running that. You are correct!

A Docker Hub search on the keyword ‘registry’ brings up the following image as the top result:

|private_registry|

Run a container from `registry` Dockerhub image

.. code-block:: bash

	$ docker run -d -p 5000:5000 --name registry registry:2

Run `docker ps -l` to check the recent container from this Docker image

.. code-block:: bash

	$ docker ps -l
	CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                    NAMES
	6e44a0459373        registry:2          "/entrypoint.sh /e..."   11 seconds ago      Up 10 seconds       0.0.0.0:5000->5000/tcp   registry

1.2.2 Tag the image that you want to push
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Next step is to tag your image under the registry namespace and push it there

.. code-block:: bash

	$ REGISTRY=localhost:5000
	$ docker tag myfirstapp $REGISTRY/$(whoami)/myfirstapp:1.0

1.2.2 Publish the image into the local registry
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Finally push the image to the local registry

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

1.2.3 Pull and run the image from the local repository
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

You can also pull the image from the local repository similar to how you pull it from Dockerhub and run a container from it

.. code-block:: bash

	docker run -d -P --name=myfirstapplocal $REGISTRY/$(whoami)/myfirstapp:1.0

2. Automated Docker image building from github
==============================================

You can also build your images automatically from a build context stored in a repository. 

`A build context is a Dockerfile and any files at a specific location. For an automated build, the build context is a repository containing a Dockerfile.`

Automated Builds have several advantages:

- Images built in this way are built exactly as specified.
- The Dockerfile is available to anyone with access to your Docker Hub repository.
- Your repository is kept up-to-date with code changes automatically.
- Automated Builds are supported for both public and private repositories on both GitHub and Bitbucket. This document guides you through the process of working with automated builds.

2.1 Prerequisites
~~~~~~~~~~~~~~~~~

To use automated builds, you first must have an account on `Docker Hub <https://hub.docker.com>`_ and on the hosted repository provider (`GitHub <https://github.com/>`_ or `Bitbucket <https://bitbucket.org/>`_). 

.. Note::

	- If you have previously linked your Github or Bitbucket account, you must have chosen the Public and Private connection type. To view your current connection settings, log in to Docker Hub and choose Profile > Settings > Linked Accounts & Services.

	- Building Windows containers is not supported.

2.2 Link to a hosted repository service
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Here are instructions for linking to a hosted repository service

1.	Log into Docker Hub.

2.	Navigate to Profile > Settings > Linked Accounts & Services.

3.	Click the service you want to link.
	The system prompts you to choose between Public and Private and Limited Access. The Public and Private connection type is required if you want to use the Automated Builds.

4.	Press Select under Public and Private connection type.
	The system prompts you to enter your service credentials (Bitbucket or GitHub) to login.

After you grant access to your code repository, the system returns you to Docker Hub and the link is complete. For example, github linked hosted repository looks like this:

|auto_build-1|

2.3 Create an automated build
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Automated build repositories rely on the integration with your code repository to build. 

Let's create an automatic build for our `flask-app` using the instructions below:

1. Initialize git repository for the `flask-app` directory

.. code-block:: bash

	$ git init
	Initialized empty Git repository in /Users/upendra_35/Documents/git.repos/flask-app/.git/
	
	$ git status
	On branch master

	Initial commit

	Untracked files:
  	(use "git add <file>..." to include in what will be committed)

		Dockerfile
		app.py
		requirements.txt
		templates/

	nothing added to commit but untracked files present (use "git add" to track) 

	$ git add * && git commit -m"Add files and folders"
	[master (root-commit) cfdf021] Add files and folders
	 4 files changed, 75 insertions(+)
	 create mode 100644 Dockerfile
	 create mode 100644 app.py
	 create mode 100644 requirements.txt
	 create mode 100644 templates/index.html

2. Create a repository on github 

3. Push the repoisotry to github

.. code-block:: bash

	$ git remote add origin https://github.com/upendrak/flask-app.git

	$ git push -u origin master
	Counting objects: 7, done.
	Delta compression using up to 8 threads.
	Compressing objects: 100% (5/5), done.
	Writing objects: 100% (7/7), 1.44 KiB | 0 bytes/s, done.
	Total 7 (delta 0), reused 0 (delta 0)
	To https://github.com/upendrak/flask-app.git
	 * [new branch]      master -> master
	Branch master set up to track remote branch master from origin.

4.	Select Create > Create Automated Build from Docker Hub.

	The system prompts you with a list of User/Organizations and code repositories.

2.	For now select from the github User.

4.	Pick the project to build. In this case `flask-app`

	The system displays the Create Automated Build dialog.

|auto_build-2|

.. Note::

	The dialog assumes some defaults which you can customize. By default, Docker builds images for each branch in your repository. It assumes the Dockerfile lives at the root of your source. When it builds an image, Docker tags it with the branch name.

5.	Customize the automated build by pressing the Click here to customize this behavior link.

|auto_build-3|

Specify which code branches or tags to build from. You can add new configurations by clicking the + (plus sign). The dialog accepts regular expressions.

6.	Click Create.

.. important::

	The first time you create a new automated build, Docker Hub builds your image. In a few minutes, you should see your new build on the image dashboard. The Build Details page shows a log of your build systems:

	During the build process, Docker copies the contents of your Dockerfile to Docker Hub. The Docker community (for public repositories) or approved team members/orgs (for private repositories) can then view the Dockerfile on your repository page.

	The build process looks for a README.md in the same directory as your Dockerfile. If you have a README.md file in your repository, it is used in the repository as the full description. If you change the full description after a build, it’s overwritten the next time the Automated Build runs. To make changes, modify the README.md in your Git repository.

.. Note:: 

	You can only trigger one build at a time and no more than one every five minutes. If you already have a build pending, or if you recently submitted a build request, Docker ignores new requests.

|auto_build-4|

Exercise 1 (10 mins)
~~~~~~~~~~~~~~~~~~~~

- Add some more cat pics to the `app.py` file
- Add, Commit and Push it to your github repo
- Trigger automatic build with a new tag (2.0) on Dockerhub
- Run an instance to make sure the new pics show up
- Share your Dockerhub link url on Slack

3. Managing data in Docker
==========================

It is possible to store data within the writable layer of a container, but there are some limitations:

- The data doesn’t persist when that container is no longer running, and it can be difficult to get the data out of the container if another process needs it.

- A container’s writable layer is tightly coupled to the host machine where the container is running. You can’t easily move the data somewhere else.

Docker offers three different ways to mount data into a container from the Docker host: **volumes**, **bind mounts**, or **tmpfs volumes**. When in doubt, volumes are almost always the right choice.

**Volumes:** Created and managed by Docker. You can create a volume explicitly using the `docker volume create` command, or Docker can create a volume during container creation. When you create a volume, it is stored within a directory on the Docker host (`/var/lib/docker/` on Linux and check for the location on mac in here https://timonweb.com/posts/getting-path-and-accessing-persistent-volumes-in-docker-for-mac/). When you mount the volume into a container, this directory is what is mounted into the container. A given volume can be mounted into multiple containers simultaneously. When no running container is using a volume, the volume is still available to Docker and is not removed automatically. You can remove unused volumes using `docker volume prune`. 

**Bind mounts:** When you use a bind mount, a file or directory on the host machine is mounted into a container. The file or directory is referenced by its full path on the host machine. The file or directory does not need to exist on the Docker host already. It is created on demand if it does not yet exist. Bind mounts are very performant, but they rely on the host machine’s filesystem having a specific directory structure available. If you are developing new Docker applications, consider using named volumes instead. You can’t use Docker CLI commands to directly manage bind mounts.

.. Warning:: 

	One side effect of using bind mounts, for better or for worse, is that you can change the host filesystem via processes running in a container, including creating, modifying, or deleting important system files or directories. This is a powerful ability which can have security implications, including impacting non-Docker processes on the host system.

**tmpfs mounts:** A tmpfs mount is not persisted on disk, either on the Docker host or within a container. It can be used by a container during the lifetime of the container, to store non-persistent state or sensitive information. For instance, internally, swarm services use tmpfs mounts to mount secrets into a service’s containers.

3.1 Volumes 
~~~~~~~~~~~

|volumes|

Volumes are often a better choice than persisting data in a container’s writable layer, because using a volume does not increase the size of containers using it, and the volume’s contents exist outside the lifecycle of a given container. While bind mounts (which we will see later) are dependent on the directory structure of the host machine, volumes are completely managed by Docker. Volumes have several advantages over bind mounts:

- Volumes are easier to back up or migrate than bind mounts.
- You can manage volumes using Docker CLI commands or the Docker API.
- Volumes work on both Linux and Windows containers.
- Volumes can be more safely shared among multiple containers.
- A new volume’s contents can be pre-populated by a container.

.. Note::

	If your container generates non-persistent state data, consider using a tmpfs mount to avoid storing the data anywhere permanently, and to increase the container’s performance by avoiding writing into the container’s writable layer.

3.1.1 Choose the -v or –mount flag for mounting volumes
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Originally, the `-v` or `--volume` flag was used for standalone containers and the `--mount` flag was used for swarm services. However, starting with Docker 17.06, you can also use `--mount` with standalone containers. In general, `--mount` is more explicit and verbose. The biggest difference is that the `-v` syntax combines all the options together in one field, while the `--mount` syntax separates them. Here is a comparison of the syntax for each flag.

.. Tip::

 	New users should use the `--mount` syntax. Experienced users may be more familiar with the `-v` or `--volume` syntax, but are encouraged to use `--mount`, because research has shown it to be easier to use.

`-v` or `--volume`: Consists of three fields, separated by colon characters (:). The fields must be in the correct order, and the meaning of each field is not immediately obvious.
- In the case of named volumes, the first field is the name of the volume, and is unique on a given host machine.
- The second field is the path where the file or directory are mounted in the container.
- The third field is optional, and is a comma-separated list of options, such as ro.

`--mount`: Consists of multiple key-value pairs, separated by commas and each consisting of a `<key>=<value>` tuple. The `--mount` syntax is more verbose than `-v` or `--volume`, but the order of the keys is not significant, and the value of the flag is easier to understand.
- The type of the mount, which can be bind, volume, or tmpfs.
- The source of the mount. For named volumes, this is the name of the volume. For anonymous volumes, this field is omitted. May be specified as source or src.
- The destination takes as its value the path where the file or directory is mounted in the container. May be specified as destination, dst, or target.
- The readonly option, if present, causes the bind mount to be mounted into the container as read-only.

.. Note::

	The `--mount` and `-v` examples have the same end result.

3.1.2. Create and manage volumes
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Unlike a bind mount, you can create and manage volumes outside the scope of any container.

Let's create a volume

.. code-block:: bash

	$ docker volume create my-vol

List volumes:

.. code-block:: bash

	$ docker volume ls

	local               my-vol

Inspect a volume by looking at the Mount section in the `docker volume inspect`

.. code-block:: bash

	$ docker volume inspect my-vol
	[
	    {
	        "Driver": "local",
	        "Labels": {},
	        "Mountpoint": "/var/lib/docker/volumes/my-vol/_data",
	        "Name": "my-vol",
	        "Options": {},
	        "Scope": "local"
	    }
	]

Remove a volume

.. code-block:: bash

	$ docker volume rm my-vol

3.1.3 Start a container with a volume
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Let'now start a container with a volume that does not yet exist (The Docker will create the volume on the fly). The following example mounts the volume `myvol2` into `/app/` in the container.

.. Note::

	The `-v` and `--mount` examples below produce the same result. You can’t run them both unless you remove the `devtest` container and the `myvol2` volume after running the first one.

.. code-block:: bash

	$ docker run -d --name devtest --mount source=myvol2,target=/app upendradevisetty/myfirstappauto:1.0

Use `docker inspect devtest` to verify that the volume was created and mounted correctly. Look for the Mounts section:

.. code-block:: bash

	$ docker inspect devtest
	"Mounts": [
	    {
	        "Type": "volume",
	        "Name": "myvol2",
	        "Source": "/var/lib/docker/volumes/myvol2/_data",
	        "Destination": "/app",
	        "Driver": "local",
	        "Mode": "",
	        "RW": true,
	        "Propagation": ""
	    }
	],

This shows that the mount is a volume, it shows the correct source and destination, and that the mount is read-write.

Stop the container and remove the volume.

.. code-block:: bash

	$ docker stop devtest

	$ docker rm devtest

	$ docker volume rm myvol2

3.1.4 Populate a volume using a container
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

If you start a container which creates a new volume, as above, and the container has files or directories in the directory to be mounted (such as `/app/` above), the directory’s contents are copied into the volume. The container then mounts and uses the volume, and other containers which use the volume also have access to the pre-populated content.

To illustrate this, this example starts an `nginx` container and populates the new volume `nginx-vol` with the contents of the container’s `/usr/share/nginx/html` directory, which is where Nginx stores its default HTML content.

.. code-block::

	$ docker run -d -p 8890:80 --name=nginxtest --mount source=nginx-vol,target=/usr/share/nginx/html nginx:latest

After running either of these examples, run the following commands to clean up the containers and volumes.

.. code-block:: bash

	$ docker stop nginxtest

	$ docker rm nginxtest

	$ docker volume rm nginx-vol

3.2 Bind mounts
~~~~~~~~~~~~~~~

|bind_mount|

If you use `--mount` to bind-mount a file or directory that does not yet exist on the Docker host, Docker does not automatically create it for you, but generates an error.

3.2.1 Start a container with a bind mount
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Consider a case where you have a directory source and that when you build the source code, the artifacts are saved into another directory `source/target/`. You want the artifacts to be available to the container at `/app/`, and you want the container to get access to a new build each time you build the source on your development host. 

Use the following command to bind-mount the target/ directory into your container at /app/. Run the command from within the source directory. The `$(pwd)` sub-command expands to the current working directory on Linux or macOS hosts.

.. code-block:: bash

	$ mkdir data

	$ docker run -d -p 8891:80 --name devtest --mount type=bind,source="$(pwd)"/data,target=/app nginx:latest

Use `docker inspect devtest` to verify that the bind mount was created correctly. Look for the "Mounts" section

.. code-block::

	$ docker inspect devtest

	"Mounts": [
	            {
	                "Type": "bind",
	                "Source": "/Users/upendra_35/Documents/git.repos/flask-app/data",
	                "Destination": "/app",
	                "Mode": "",
	                "RW": true,
	                "Propagation": "rprivate"
	            }
	        ],

This shows that the mount is a bind mount, it shows the correct source and target, it shows that the mount is read-write, and that the propagation is set to rprivate.

Stop the container:

.. code-block:: bash

	$ docker rm -f devtest

3.2.2 Use a read-only bind mount
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

For some development applications, the container needs to write into the bind mount, so changes are propagated back to the Docker host. At other times, the container only needs read access.

This example modifies the one above but mounts the directory as a read-only bind mount, by adding `ro` to the (empty by default) list of options, after the mount point within the container. Where multiple options are present, separate them by commas.

.. code-block:: bash

	$ docker run -d -p 8891:80 --name devtest --mount type=bind,source="$(pwd)"/data,target=/app,readonly nginx:latest

Use `docker inspect devtest` to verify that the bind mount was created correctly. Look for the Mounts section:

.. code-block:: bash

	$ docker inspect devtest

	"Mounts": [
	                {
	                    "Type": "bind",
	                    "Source": "/Users/upendra_35/Documents/git.repos/flask-app/data",
	                    "Target": "/app",
	                    "ReadOnly": true
	                }
	            ]

Stop the container:

.. code-block:: bash

	$ docker rm -f devtest

3.3 tmpfs
~~~~~~~~~

|tmpfs|

Volumes and bind mounts are mounted into the container’s filesystem by default, and their contents are stored on the host machine.
There may be cases where you do not want to store a container’s data on the host machine, but you also don’t want to write the data into the container’s writable layer, for performance or security reasons, or if the data relates to non-persistent application state. An example might be a temporary one-time password that the container’s application creates and uses as-needed. To give the container access to the data without writing it anywhere permanently, you can use a tmpfs mount, which is only stored in the host machine’s memory (or swap, if memory is low). When the container stops, the tmpfs mount is removed. If a container is committed, the tmpfs mount is not saved.

.. code-block:: bash

	$ docker run -d -p 8891:80 --name devtest --mount type=tmpfs,target=/app nginx:latest

Use `docker inspect devtest` to verify that the bind mount was created correctly. Look for the Mounts section:

.. code-block:: bash

	$ docker inspect devtest

	"Mounts": [
	            {
	                "Type": "tmpfs",
	                "Source": "",
	                "Destination": "/app",
	                "Mode": "",
	                "RW": true,
	                "Propagation": ""
	            }
	        ],

Stop the container:

.. code-block:: bash

	$ docker rm -f devtest

4. Docker Compose for multi container apps
==========================================

Docker Compose is a “tool for defining and running your multi-container Docker applications”. 

Main advantages of Docker compose include:

- Your applications can be defined in a YAML file where all the options that you used in `docker run` are now defined (Reproducibility).
- Docker Compose allows you to manage your application as a single entity rather than dealing with individual containers (Simplicity).

Let's now convert create a simple web app with Docker Compose and Flask (which you already seen before) using Redis (we end up with a Flask container and a Redis container all on one host)

1. You’ll need a directory for your project on your host machine:

.. code-block:: bash

	$ mkdir compose_flask && cd compose_flask

2. Add the following to requirements.txt inside `compose_flask` directory:

.. code-block:: bash

	flask
	redis

3. Copy and paste the following code into a new file called `app.py` inside `compose_flask` directory:

.. code-block:: bash

	# compose_flask/app.py
	from flask import Flask
	from redis import Redis

	app = Flask(__name__)
	redis = Redis(host='redis', port=6379)

	@app.route('/')
	def hello():
	    redis.incr('hits')
	    return 'This Compose/Flask demo has been viewed %s time(s).' % redis.get('hits')

	if __name__ == "__main__":
	    app.run(host="0.0.0.0", debug=True)

4. Add the following code to a new file, docker-compose.yml, in your project directory:

.. code-block:: bash

	version: '2'
	services:
	    web:
	        restart: always
	        build: .
	        ports:
	            - "8888:5000"
	        volumes:
	            - .:/code
	        depends_on:
	            - redis
	    redis:
	        restart: always
	        image: redis

.. Note:: 

	- `restart: always` means that it will restart whenever it fails
	- We define two services, web and redis.
	- The web service builds from the Dockerfile in the current directory.
	- Forwards the container’s exposed port (5000) to port 8888 on the host.
	- Mounts the project directory on the host to /code inside the container (allowing you to modify the code without having to rebuild the image).
	- And links the web service to the Redis service.
	- The redis service uses the latest Redis image from Docker Hub.

5. Build and Run with `docker-compose up -d` command

.. code-block:: bash

	$ docker-compose up -d

	Building web
	Step 1/5 : FROM python:2.7
	2.7: Pulling from library/python
	f49cf87b52c1: Already exists
	7b491c575b06: Already exists
	b313b08bab3b: Already exists
	51d6678c3f0e: Already exists
	09f35bd58db2: Already exists
	f7e0c30e74c6: Pull complete
	c308c099d654: Pull complete
	339478b61728: Pull complete
	Digest: sha256:8cb593cb9cd1834429f0b4953a25617a8457e2c79b3e111c0f70bffd21acc467
	Status: Downloaded newer image for python:2.7
	 ---> 9e92c8430ba0
	Step 2/5 : ADD . /code
	 ---> 746bcecfc3c9
	Step 3/5 : WORKDIR /code
	 ---> c4cf3d6cb147
	Removing intermediate container 84d850371a36
	Step 4/5 : RUN pip install -r requirements.txt
	 ---> Running in d74c2e1cfbf7
	Collecting flask (from -r requirements.txt (line 1))
	  Downloading Flask-0.12.2-py2.py3-none-any.whl (83kB)
	Collecting redis (from -r requirements.txt (line 2))
	  Downloading redis-2.10.6-py2.py3-none-any.whl (64kB)
	Collecting itsdangerous>=0.21 (from flask->-r requirements.txt (line 1))
	  Downloading itsdangerous-0.24.tar.gz (46kB)
	Collecting Jinja2>=2.4 (from flask->-r requirements.txt (line 1))
	  Downloading Jinja2-2.10-py2.py3-none-any.whl (126kB)
	Collecting Werkzeug>=0.7 (from flask->-r requirements.txt (line 1))
	  Downloading Werkzeug-0.14.1-py2.py3-none-any.whl (322kB)
	Collecting click>=2.0 (from flask->-r requirements.txt (line 1))
	  Downloading click-6.7-py2.py3-none-any.whl (71kB)
	Collecting MarkupSafe>=0.23 (from Jinja2>=2.4->flask->-r requirements.txt (line 1))
	  Downloading MarkupSafe-1.0.tar.gz
	Building wheels for collected packages: itsdangerous, MarkupSafe
	  Running setup.py bdist_wheel for itsdangerous: started
	  Running setup.py bdist_wheel for itsdangerous: finished with status 'done'
	  Stored in directory: /root/.cache/pip/wheels/fc/a8/66/24d655233c757e178d45dea2de22a04c6d92766abfb741129a
	  Running setup.py bdist_wheel for MarkupSafe: started
	  Running setup.py bdist_wheel for MarkupSafe: finished with status 'done'
	  Stored in directory: /root/.cache/pip/wheels/88/a7/30/e39a54a87bcbe25308fa3ca64e8ddc75d9b3e5afa21ee32d57
	Successfully built itsdangerous MarkupSafe
	Installing collected packages: itsdangerous, MarkupSafe, Jinja2, Werkzeug, click, flask, redis
	Successfully installed Jinja2-2.10 MarkupSafe-1.0 Werkzeug-0.14.1 click-6.7 flask-0.12.2 itsdangerous-0.24 redis-2.10.6
	 ---> 5cc574ff32ed
	Removing intermediate container d74c2e1cfbf7
	Step 5/5 : CMD python app.py
	 ---> Running in 3ddb7040e8be
	 ---> e911b8e8979f
	Removing intermediate container 3ddb7040e8be
	Successfully built e911b8e8979f
	Successfully tagged composeflask_web:latest

And that’s it! You should be able to see the Flask application running on http://localhost:8888

|docker-compose|

The code for the above compose example is available `here <https://github.com/upendrak/compose_flask>`_

Exercise 2 (10 mins)
~~~~~~~~~~~~~~~~~~~~

- Customize webpage by using a template
- Create a automatic build for `compose-flask` project directory
- Share your Dockerhub link url on Slack

5. Improving your data science workflow using Docker containers
===============================================================
Containers are lightweight versions of traditional virtual machines. They don’t take up large amounts of space on your server, they are easy to create and destroy, and they are fast to boot up. They also make creating repeatable data science environments easy.

For a data scientist, running a container that is already equipped with the libraries and tools needed for a particular analysis eliminates the need to spend hours debugging packages across different environments or configuring custom environments.

We will see few examples of launching isolated Jupyter and RStudio sessions outfitted with their choice of libraries and tools.

Why Set Up a Data Science Environment in a Container?

One reason is speed. We want data scientists using our platform to launch a Jupyter or RStudio session in minutes, not hours. We also want them to have that fast user experience while still working in a governed, central architecture (rather than on their local machines). The process of getting an environment up and running varies from company to company, but in some cases, a data scientist must submit a formal request to IT and wait for days or weeks, depending on the backlog. That puts a burden on both groups.

Containerization benefits both data science and IT/technical operations teams. In the DataScience.com Platform, for instance, we allow IT to configure environments with different languages, libraries, and settings in an admin dashboard and make those images available in the dropdown menu when a data scientist launches a session. These environments can be selected for any run, session, scheduled job, or API. (Or you don’t have to configure anything at all. We provide plenty of standard environment templates to choose from.)

Ultimately, containers solve a lot of common problems associated with doing data science work at the enterprise level. They take the pressure off of IT to produce custom environments for every analysis, standardize how data scientists work, and ensure that old code doesn’t stop running because of environment changes. To start using containers and our library of curated images to do collaborative data science work, request a demo of our platform today.

Configuring a data science environment can be a pain. Dealing with inconsistent package versions, having to dive through obscure error messages, and having to wait hours for packages to compile can be frustrating. This makes it hard to get started with data science in the first place, and is a completely arbitrary barrier to entry.

The past few years have seen the rise of technologies that help with this by creating isolated environments. We'll be exploring one in particular, Docker. Docker makes it fast and easy to create new data science environments, and use tools such as Jupyter notebooks to explore your data.

With Docker, we can download an image file that contains a set of packages and data science tools. We can then boot up a data science environment using this image within seconds, without the need to manually install packages or wait around. This environment is called a Docker container. Containers eliminate configuration problems -- when you start a Docker container, it has a known good state, and all the packages work properly.

7. Deploy your app
==================

.. |docker_image| image:: ../img/docker_image.png
  :width: 750
  :height: 700 

.. |private_registry| image:: ../img/private_registry.png
  :width: 750
  :height: 700 

.. |auto_build-1| image:: ../img/auto_build-1.png
  :width: 750
  :height: 700 

.. |auto_build-2| image:: ../img/auto_build-2.png
  :width: 750
  :height: 700 

.. |auto_build-3| image:: ../img/auto_build-3.png
  :width: 750
  :height: 700 

.. |auto_build-4| image:: ../img/auto_build-4.png
  :width: 750
  :height: 700 

.. |volumes| image:: ../img/volumes.png
  :width: 750
  :height: 700 

.. |bind_mount| image:: ../img/bind_mount.png
  :width: 750
  :height: 700 

.. |tmpfs| image:: ../img/tmpfs.png
  :width: 750
  :height: 700 

.. |docker-compose| image:: ../img/dc-1.png
  :width: 750
  :height: 700 