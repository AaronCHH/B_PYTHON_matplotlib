
# Big Data

<!-- toc orderedList:0 depthFrom:1 depthTo:6 -->

- [Big Data](#big-data)
	- [Distributed Data](#distributed-data)
		- [Disco](#disco)
		- [Disco Clusters on AWS](#disco-clusters-on-aws)
		- [Local Hadoop](#local-hadoop)

<!-- tocstop -->

*The Expurgated Content*

The following sections were removed from the notebook for the "Big Data" chapter due to the following:
 1. Running low on time
 1. Disco set up on Amazon EC2 in coordinated cluster mode being rather more involved than running Disco on a local machine.
 1. Hadoop on EMR is covered in the main notebook, and the usefulness of a tiny section on a local machine running Hadoop seem rather limited (in fact, only the first few lines were created in this section).

However, it is felt that the content may be useful for some, so it is preserved in this orphaned notebook.

## Distributed Data

### Disco

The [Disco project](http://discoproject.org/) is an implementation of mapreduce which is generally much easier to set up than Hadoop. If you don't need the ecosystm of JVM libraries that has been created around Hadoop and want to do distributed jobs across a cluster in with a quick installation, then Disco may be for you. Disco can be used for a variety data mining tasks: large-scale analytics, building probabilistic models, and full-text indexing the Web, just to name a few examples.

Disco supports parallel computations over large data sets, stored on an unreliable cluster of computers, as in the original framework created by Google. This makes it a perfect tool for analyzing and processing large data sets, without having to worry about difficult technicalities related to distribution such as communication protocols, load balancing, locking, job scheduling, and fault tolerance, which are handled by Disco.

The Disco core is written in Erlang, a functional language that is designed for building robust, fault-tolerant, distributed applications. However, in most casese, you won't even know that Erlang is there. Users of Disco typically write jobs in Python, with all the benefits that brings.

We've provided some ``Dockerfile``s for building the necessary images. Let's get started by building the base Docker image for Disco which sets up the Erlang dependency:

```bash
$ docker build -t masteringmatplotlib/erlang ./docker/erlang
```

And make sure it works:

```erlang
$ docker run -i -t masteringmatplotlib/erlang
Erlang/OTP 17 [erts-6.2] [source] [64-bit] [smp:8:8] [async-threads:10] [hipe] [kernel-poll:false]

Eshell V6.2  (abort with ^G)
1> io:format("Testing ...~n").
Testing ...
ok
2>
```

Now we can build the image for the disco server, which will use the one we just built:

```bash
$ docker build -t masteringmatplotlib/disco ./docker/disco
```

Here's what that ``Dockerfile`` looks like:


```
cat ../docker/disco/Dockerfile
```

    FROM masteringmatplotlib/erlang
    MAINTAINER Duncan McGreggor <oubiwann@gmail.com>
    ENV ORG discoproject
    ENV REPO disco
    ENV DISCO_USER disco
    ENV HOME /home/$DISCO_USER
    ENV DISCO_HOME $HOME/$REPO
    ENV RELEASE 0.5.4
    ENV PYTHONPATH /usr/lib/python3.4/site-packages:$PYTHONPATH
    RUN apt-get update
    RUN apt-get -y upgrade
    RUN apt-get install -y -q openssh-server
    RUN ln -s `which python3` `dirname $(which python3)`/python

    # setup SSH for root user
    RUN ssh-keygen -N '' -f /root/.ssh/id_dsa
    RUN cat /root/.ssh/id_dsa.pub >> /root/.ssh/authorized_keys
    RUN echo -n "localhost " > /root/.ssh/known_hosts
    RUN cat /etc/ssh/ssh_host_rsa_key.pub >> /root/.ssh/known_hosts

    # setup SSH for disco user
    RUN adduser --system $DISCO_USER --shell /bin/sh
    RUN mkdir /home/$DISCO_USER/.ssh
    RUN ssh-keygen -N '' -f /home/$DISCO_USER/.ssh/id_dsa && \
        cat /home/$DISCO_USER/.ssh/id_dsa.pub >> \
            /home/$DISCO_USER/.ssh/authorized_keys && \
        echo -n "localhost " > /home/$DISCO_USER/.ssh/known_hosts && \
        cat /etc/ssh/ssh_host_rsa_key.pub >> \
            /home/$DISCO_USER/.ssh/known_hosts && \
        chown $DISCO_USER -R /home/$DISCO_USER/.ssh
    RUN mkdir -p /usr/var/disco/data && \
        chown -R disco /usr/var/disco

    # install disco from git clone
    RUN cd $HOME && \
        git clone https://github.com/$ORG/$REPO.git && \
        cd $REPO && \
        git checkout tags/$RELEASE && \
        make && make install
    RUN echo "defaultcookievalue" > $HOME/.erlang.cookie && \
        chmod 400 $HOME/.erlang.cookie
    RUN chown -R $DISCO_USER $HOME

    RUN echo '#!/bin/sh' > $HOME/start.sh
    RUN echo "/etc/init.d/ssh start" >> $HOME/start.sh
    RUN echo "su disco -c '$DISCO_HOME/bin/disco nodaemon'" >> $HOME/start.sh
    RUN chmod 755 $HOME/start.sh

    EXPOSE 22/tcp 8989/tcp 8990/tcp 999/tcp

    CMD $HOME/start.sh


Once Disco starts, we're going to want to access the Disco HTTP admin interface; with Linux, this should be no problem, but on Mac OS X you will need to set up an SSH tunnel through boot2docker:

```bash
$ boot2docker ssh -L localhost:8989:localhost:8989
                        ##        .
                  ## ## ##       ==
               ## ## ## ##      ===
           /""""""""""""""""\___/ ===
      ~~~ {~~ ~~~~ ~~~ ~~~~ ~~ ~ /  ===- ~~~
           \______ o          __/
             \    \        __/
              \____\______/
 _                 _   ____     _            _
| |__   ___   ___ | |_|___ \ __| | ___   ___| | _____ _ __
| '_ \ / _ \ / _ \| __| __) / _` |/ _ \ / __| |/ / _ \ '__|
| |_) | (_) | (_) | |_ / __/ (_| | (_) | (__|   <  __/ |
|_.__/ \___/ \___/ \__|_____\__,_|\___/ \___|_|\_\___|_|
boot2docker: 1.3.0
             master : a083df4 - Thu Oct 16 17:05:03 UTC 2014
docker@boot2docker:~$
```

You can leave that terminal window, and work in another.



Let's make sure our new image works:

```bash
$ docker run -i -t masteringmatplotlib/disco-server
```

You should see output like the following:

```bash
 * Starting OpenBSD Secure Shell server sshd                                                                                                                                                                                          [ OK ]
Erlang/OTP 17 [erts-6.2] [source] [64-bit] [smp:8:8] [async-threads:10] [hipe] [kernel-poll:true]

Eshell V6.2  (abort with ^G)
(disco_8989_master@7f54072a7f32)1> 17:04:41.152 [info] Application lager started on node disco_8989_master@7f54072a7f32
17:04:41.267 [info] Application inets started on node disco_8989_master@7f54072a7f32
17:04:41.268 [info] DISCO BOOTS
17:04:41.272 [info] Disco proxy disabled
17:04:41.276 [info] DDFS master starts
17:04:41.283 [info] Event server starts
17:04:41.287 [info] Disco config starts
17:04:41.293 [info] DISCO SERVER STARTS
17:04:41.296 [info] Fair scheduler starts
17:04:41.297 [info] Scheduler uses fair policy
17:04:41.301 [info] Fair scheduler: Fair policy
17:04:41.308 [info] Config table updated
17:04:41.456 [info] Starting node "disco_8989_slave" on "localhost" ("localhost")
17:04:41.466 [info] web server (mochiweb) starts
17:04:41.467 [info] Application disco started on node disco_8989_master@7f54072a7f32
17:04:42.319 [info] lock_server starts on disco_8989_slave@localhost
17:04:42.328 [info] ddfs_node initialized on disco_8989_master@7f54072a7f32 with volumes: ["vol0"]
17:04:42.328 [info] ddfs_node initialized on disco_8989_slave@localhost with volumes: ["vol0"]
17:04:42.331 [info] Tempgc: error listing "/usr/var/disco/data/localhost": {error,enoent}
17:04:42.331 [info] Tempgc: one pass completed on disco_8989_slave@localhost
17:04:42.336 [info] ddfs_node starts on disco_8989_master@7f54072a7f32
17:04:42.336 [info] Node started at disco_8989_slave@localhost (reporting as disco_8989_master@7f54072a7f32) on "localhost"
17:04:42.350 [info] Started ddfs_put at disco_8989_slave@localhost on port 8990
17:04:42.351 [info] ddfs_node starts on disco_8989_slave@localhost
17:04:42.472 [info] Node started at disco_8989_slave@localhost (reporting as disco_8989_slave@localhost) on "localhost"
```

### Disco Clusters on AWS

Configure the ``aws`` CLI tool with your info and a default region:
```
$ aws configure
AWS Access Key ID [None]: YOURAWSACCESSKEYID
AWS Secret Access Key [None]: YOURAWSSECRETACCESSKEY
Default region name [None]: us-west-2
Default output format [None]: text
```

We will be using the AWS EC2 Container Service, or *ECS*. You should read the [documentation for ECS](http://docs.aws.amazon.com/AmazonECS/latest/developerguide/get-set-up-for-amazon-ecs.html) which shows how to get setup for ECS, including:
 * Creating an IAM policy for your container instances
 * Creating an IAM role for your container instances
 * Creating a Virtual Private Cloud for your container instances

If you do not have these set up, ECS will not work. Be sure to follow their instructions carefully.

Once set up, let's create an ECS cluster:

```
$ aws ecs create-cluster
CLUSTER	arn:aws:ecs:us-west-2:149557551438:cluster/default	default	ACTIVE
```

Now we need to launch container instances. Go to the AWS EC2 Console, start the "Launch Instance" wizard, and in the "Community APIs, search for "amazon-ecs-optimized" -- go ahead an launch an instance of this. Once it's running, you can execute the following command to list the container instances for the default cluster you created above:

```
$ aws ecs list-container-instances --cluster default
```
```
CONTAINERINSTANCEARNS	arn:aws:ecs:us-west-2:149557551438:container-instance/8d93c567-5cda-44b4-aad5-2bbaa6926d97
CONTAINERINSTANCEARNS	arn:aws:ecs:us-west-2:149557551438:container-instance/c537beae-be52-4ae3-aefd-93bdf9ef9e8e
```

```
$ aws ecs describe-container-instances --cluster default \
    --container-instances 8d93c567-5cda-44b4-aad5-2bbaa6926d97
```
```
CONTAINERINSTANCES	True	arn:aws:ecs:us-west-2:149557551438:container-instance/8d93c567-5cda-44b4-aad5-2bbaa6926d97	i-8309d275	ACTIVE
REGISTEREDRESOURCES	0.0	2048	0	CPU	INTEGER
REGISTEREDRESOURCES	0.0	7483	0	MEMORY	INTEGER
REGISTEREDRESOURCES	0.0	0	0	PORTS	STRINGSET
STRINGSETVALUE	2376
STRINGSETVALUE	22
STRINGSETVALUE	51678
STRINGSETVALUE	2375
REMAININGRESOURCES	0.0	2048	0	CPU	INTEGER
REMAININGRESOURCES	0.0	7483	0	MEMORY	INTEGER
REMAININGRESOURCES	0.0	0	0	PORTS	STRINGSET
STRINGSETVALUE	2376
STRINGSETVALUE	22
STRINGSETVALUE	51678
STRINGSETVALUE	2375
```

Next, create a file called ``disco-manager.json`` with the following contents:

```json
{
    "family": "disco-manager",
    "containerDefinitions": [
        {
            "environment": [],
            "name": "disco-manager",
            "image": "masteringmatplotlib/disco",
            "cpu": 20,
            "memory": 2000,
            "portMappings": [
                {
                    "containerPort": 8989,
                    "hostPort": 8989
                },
                {
                    "containerPort": 8990,
                    "hostPort": 8990
                },
                {
                    "containerPort": 8999,
                    "hostPort": 8999
                }
            ],
            "essential": true
        }
    ]
}
```

Next, create a file called ``disco-worker.json`` with the following contents:

```json
{
    "family": "disco-worker",
    "containerDefinitions": [
        {
            "name": "disco-worker",
            "image": "masteringmatplotlib/disco",
            "cpu": 10,
            "memory": 1000,
            "portMappings": [
                {
                    "containerPort": 8989,
                    "hostPort": 8989
                },
                {
                    "containerPort": 8990,
                    "hostPort": 8990
                },
                {
                    "containerPort": 8999,
                    "hostPort": 8999
                }
            ],
            "essential": true
        }
    ]
}
```

Then register your Disco task:

```
$ aws ecs register-task-definition --cli-input-json file://disco-manager.json
$ aws ecs register-task-definition --cli-input-json file://disco-worker.json
```

Let's make sure they've been registered:

```
$ aws ecs list-task-definitions
TASKDEFINITIONARNS	arn:aws:ecs:us-west-2:149557551438:task-definition/disco-manager:1
TASKDEFINITIONARNS	arn:aws:ecs:us-west-2:149557551438:task-definition/disco-worker:1
```

With the container instances running, we can run our task:

```
$ aws ecs run-task --task-definition disco-manager
```

### Local Hadoop

* Download and install Hadoop for your operating system (e.g., [part 1](http://amodernstory.com/2014/09/23/installing-hadoop-on-mac-osx-yosemite/) and [part2](http://amodernstory.com/2014/09/23/hadoop-on-mac-osx-yosemite-part-2/) of a tutorial for Mac OS X and a [tutorial for Ubuntu](http://www.michael-noll.com/tutorials/running-hadoop-on-ubuntu-linux-single-node-cluster/)).
* Start the HDFS

Create a directory on Hadoop file system:

```
$ hdfs dfs -mkdir /weather
```

Save some our generated CSV data to HDFS:

```
$ hdfs dfs -put data/{0,1,2}.csv /weather
```
