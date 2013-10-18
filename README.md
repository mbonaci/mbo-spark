Getting started with Spark
-----

This tutorial was written in _October 2013._  
At the time, the current development version of Spark was 0.9.0.  

The tutorial covers Spark setup on Ubuntu 12.04:
- installation of all Spark prerequisites
- Spark installation
- basic Spark configuration
- standalone cluster setup (one master and 4 slaves on a single machine)
- running the PI (3.14) approximation job on a standalone cluster

Next part of this (soon to be) series will cover Spark setup in Eclipse.

### My setup
_Before installing Spark:_
- Ubuntu 12.04 LTS 32-bit
- OpenJDK 1.6.0_27
- Scala 2.9.3
- Maven 3.0.4
- Python 2.7.3 (you already have this)
- Git 1.7.9.5  (and this, I presume)
  
  
The official one-liner describes Spark as "a general purpose cluster computing platform".  

Spark was conceived and developed at Berkeley labs. It is currently incubated at Apache and improved and maintained by a rapidly growing community of users, thus it's expected to graduate to top-level project very soon.  
It's written mainly in Scala, and provides Scala, Java and Python APIs.  
Scala API provides a way to write concise, higher level routines that effectively manipulate distributed data.  

Here's a quick example of how straightforward it is to distribute some arbitrary data with Scala API:

```scala
// parallelized collection example
// example used Scala interpreter (also called 'repl') as an interface to Spark

// declare scala array:
scala> val data = Array(1, 2, 3, 4, 5)

// Scala interpreter responds with:
data: Array[Int] = Array(1, 2, 3, 4, 5)

// distribute the array in a cluster:
scala> val distData = sc.parallelize(data)

// sc is an instance of SparkCluster that's initialized for you by Scala repl

// returns Resilient Distributed Dataset:
distData: spark.RDD[Int] = spark.ParallelCollection@88eebe8e  // repl output
```

## What the hell is RDD?
**Resilient Distributed Dataset** is a collection that has been distributed _all over_ the Spark cluster. RDDs' main purpose is to support higher-level, parallel operations on data in a straightforward manner.  
There are currently two types of RDDs: **parallelized collections**, which take an existing Scala collection and run operations on it in parallel, and **Hadoop** datasets, which run functions on each record of a file in _HDFS_ (or any other storage system supported by _Hadoop_).

## Prereqs
First, let's get the requirements out of the way.  

### Installing prereqs

**Install oracle-java-6**  
Since Oracle, after they acquired Java from Sun, changed license agreement (not in a good way), Canonical only provides PPAs (_personal package archive_) for OpenJRE/OpenJDK. Since we need oracle's version of java (we probably don't, but I had some weird problems while building Spark with OpenJDK - presumably due to the fact that open-jdk places jars in `/usr/share/java`, I'm not really sure, but installation of oracle-java effectively solved all those problems), we'll need to install it and set it to be the default JVM on our system.  

Here's how:

```sh
# you may or may not want to remove open-jdk (not necessary):
$ sudo apt-get purge openjdk*

# to add PPA source repository to apt:
$ sudo add-apt-repository ppa:webupd8team/java

# to refresh the sources list:
$ sudo apt-get update

# to install JDK 6:
$ sudo apt-get install oracle-java6-installer

# to make JDK6 default JVM provider:
$ sudo apt-get install oracle-java6-set-default
```

**Check/set JAVA_HOME**

```sh
# check default java version
# if needed, reconfigure by selecting another number
sudo update-alternatives --config java
# this sets the target of the `java` symbolic link in /etc/alternatives/
# which you can confirm with:
file `which java`

# for good measure, I like to set JAVA_HOME explicitly (Spark checks for its existence)
echo "export JAVA_HOME=/usr/lib/jvm/java-6-oracle/" >> ~/.bashrc

# to reload .bashrc
source ~/.bashrc

# try it out (should return /usr/lib/jvm/java-6-oracle/)
echo $JAVA_HOME
```

- Having problems with Java setup? [Check the latest Ubuntu Java documentation](https://help.ubuntu.com/community/Java).

**Install Scala**
- download 2.9.3 binaries for your OS from [here](http://www.scala-lang.org/download/2.9.3.html). Debian/Ubuntu [direct download link](http://www.scala-lang.org/files/archive/scala-2.9.3.deb)
- to install Scala `deb`, simply fire in terminal: 

```sh
sudo dpkg -i scala-2.9.3.deb
```

**Check/set SCALA_HOME**

```sh
# append a line in .bashrc
echo "export SCALA_HOME=/usr/share/java/" >> ~/.bashrc

# to reload .bashrc just execute bash
source ~/.bashrc
```

**Install Maven**
_Skip if you'll choose to install Spark from binaries (see the next section for more info)_
This one is dead simple: 

```sh
sudo apt-get install maven
```

Be warned, a large download will take place.  
It is flat out awful that a dependency & build management tool may become so bloated that it weighs this much, but that's a whole different story...

## Installing Spark
We'll try, like a couple of hoodlums, to build the cutting edge, development version of Spark ourselves. Screw binaries, right :)  
  
If you were to say that this punk move will just complicate things, you wouldn't be far from truth. So, if you want to simplify things a bit, and avoid possible dev-version bugs down the road, go ahead and download (and install) Spark binaries from [here](http://spark.incubator.apache.org/downloads.html). If, on the other hand, you're not scared, keep following instructions. Just kidding. No, honestly, I really suggest that you use binaries (and skip to [OMG! section](#omg-i-have-a-running-spark-in-my-home)).

> Back to the story, so my grandma was building the Spark from source the other day and she noticed a couple of build errors...

- to get the Spark source code:

```sh
# clone the development repo
git clone git://github.com/apache/incubator-spark.git

# rename the folder
mv incubator-spark/ spark

# go into it
cd spark
```

- to build Spark, we'll have to use either Maven:

> But what these params bellow actually mean?
> Spark will build against Hadoop 1.0.4 by default, so if you want to read from HDFS (optional), use the correct version of Hadoop. If not, choose any version.
> For more options, take a look [here](http://spark.incubator.apache.org/docs/latest/building-with-maven.html).

```sh
# with Maven
mvn -Phadoop2-yarn -Dhadoop.version=2.0.5-alpha -Dyarn.version=2.0.5-alpha -DskipTests clean package
```

- or **sbt** (Scala build tool):

```sh
# with sbt
sbt/sbt assembly
```

Since I have successfully built Spark with _mvn_ I have never used _sbt_, so you're on your own there

## OMG! I have a running Spark in my home

> ... IN MY HOME, in my bedroom, where my wife sleeps! Where my children come to play with their toys. In my home.

Yes, that was my general feeling when I first launched a Spark master

![spark-repl](https://raw.github.com/mbonaci/mbo-spark/master/resources/spark-repl.png)

### How to get there

OK, we installed all the prerequisites and successfully built Spark. Now it's finally time to have some fun with it.

To get the ball rolling, start the Spark master:

```sh
# to start the Spark master on your localhost:
./bin/start-master.sh

# outputs master's class and log file info:
starting org.apache.spark.deploy.master.Master, logging to /home/mbo/spark/bin/../
logs/spark-mbo-org.apache.spark.deploy.master.Master-1-mbo-ubuntu-vbox.out
```

if you need more info on how the startup process works, take a look [here](http://spark.incubator.apache.org/docs/latest/spark-standalone.html).

To check out Spark web console, open http://localhost:8080/

![master's web console](https://raw.github.com/mbonaci/mbo-spark/master/resources/spark-web-console.gif)

### Starting slave workers

> Our beloved master is lonely, with no one to boss around. Let's fix that.

Spark _master_ requires passwordless `ssh` login to its _slaves_, and since we're building a standalone Spark cluster (master and all slaves on a single machine), we'll need to facilitate _localhost to localhost_ passwordless connection.

If your private key (usually `~/.ssh/rsa_id`) has a password, follow [these instructions](http://askubuntu.com/a/296574/116447) to get rid of it for connections from _localhost_.

Next, we'll need to get `openssh server` up and running:

```sh
# to install openssh-server (Ubuntu comes only with ssh client)
sudo apt-get install openssh-server

# then check whether the openssh server is started (should be)
service ssh status

# if not
service ssh start
```

### Actually starting slave workers

To tell Spark to run 4 workers on each slave machine, we'll create a new config file:

```sh
# create spark-env.sh file using a provided template
cp ./conf/spark-env.sh.template ./conf/spark-env.sh

# append a configuration param to the end of the file
echo "export SPARK_WORKER_INSTANCES=4" >> ./conf/spark-env.sh
```

Now we've hopefully prepared everything so we can finally launch 4 slave workers, on the same machine where our master is already lurking around (slaves will be assigned with random ports, thus avoiding port collision)

```sh
# to start slave workers on your localhost:
./bin/start-slaves.sh
```

You should see something similar to this:

![4 slaves start](https://raw.github.com/mbonaci/mbo-spark/master/resources/spark-4-slaves-started.png)

If you now refresh master's web console, you should see 4 slaves listed there:

![4 slaves in web console](https://raw.github.com/mbonaci/mbo-spark/master/resources/spark-master-web-console-4-slaves.gif)

Clicking on a slave's link opens its web console:

![slave web console](https://raw.github.com/mbonaci/mbo-spark/master/resources/spark-slaves-web-console.gif)

### Starting and stopping the whole cluster

Since you're going to start and stop master and slaves multiple times, let's quickly go through the simplified procedure of starting and stopping the cluster with one command:

```sh
# first, let's stop the master and all the slaves:
./bin/stop-all.sh

# then, to start all of them in one go:
./bin/start-all.sh
```

## Configuring _log4j_
I regularly configure log4j to write to a log file, instead of a console.
If you'd like to setup the same thing yourself, you'll need to modify `log4j.properties` located in `spark/conf/` directory like this:

```sh
# Initialize root logger
log4j.rootLogger=INFO, FILE
# Set everything to be logged to the console
log4j.rootCategory=INFO, FILE

# Ignore messages below warning level from Jetty, because it's a bit verbose
log4j.logger.org.eclipse.jetty=WARN

# Set the appender named FILE to be a File appender
log4j.appender.FILE=org.apache.log4j.FileAppender

# Change the path to where you want the log file to reside
log4j.appender.FILE.File=/mbo/spark/logs/SparkOut.log

# Prettify output a bit
log4j.appender.FILE.layout=org.apache.log4j.PatternLayout
log4j.appender.FILE.layout.ConversionPattern=%d{yy/MM/dd HH:mm:ss} %p %c{1}: %m%n
```

To assure that Spark will be able to find `log4j.properties`, I suggest you create a new `java-opts` file in `spark/conf/` directory and place a single line inside:

```sh
# tell Spark where log4j configuration is (modify file path, of course)
-Dlog4j.configuration=file:///mbo/spark/conf/log4j.properties
```

FYI, the same thing can be accomplished using `SPARK_JAVA_OPTS` param in `spark-env.sh`.

## Playing with Spark

Now we are ready for our first interactive session with Spark.
To launch Spark Scala interpreter:

```sh
## launch scala repl
MASTER=spark://localhost:7077 ./spark-shell
```

![spark repl](https://raw.github.com/mbonaci/mbo-spark/master/resources/spark-repl.png)

### Hello Spark

As a demonstration, let's use our Spark cluster to approximate _PI_ using [MapReduce](http://hadoop.apache.org/docs/stable/mapred_tutorial.html):  

```scala
/* throwing darts and examining coordinates */
val NUM_SAMPLES = 100000
val count = spark.parallelize(1 to NUM_SAMPLES).map(i =>
  val x = Math.random
  val y = Math.random
  if (x * x + y * y < 1) 1.0 else 0.0
).reduce(_ + _)

println("Pi is roughly " + 4 * count / NUM_SAMPLES)
```

If you're not comfortable with Scala, I recently wrote a [Java developer's Scala cheat sheet](http://mbonaci.github.io/scala/) (based on Programming in Scala SE book, by Martin Odersky, whose first edition is freely available [online](http://www.artima.com/pins1ed/)), which is basically a big reference card, where you can look up almost any Scala topic you come across.

## What Spark uses under the hood?

Since you're still reading this, I'll go ahead and assume that you're like me. You don't want to **just use** something, you like to know what you're working with and how it all works together.  
..
So let me present you with the list of software (all open source) that was installed automatically, alongside Spark (I wrote a short explanation besides ones that were unknown to me):

### My setup - _post festum_
- Hadoop 2.0.5-alpha (with Yarn)
- Zookeeper 3.4.5
- Avro 1.7.4
- Akka 2.0.5
- Lift-json 2.5
- Twitter Chill 2.9.3 (Kryo serialization for Scala)
- Jackson 2.1 (REST)
- Google Protobuf 2.4.1
- Scala-arm (Scala automatic resource management)
- Objenesis 1.2 (DI & serialization)
- Jetty 7.6.8
- ASM 4.0 (bytecode manipulation)
- Minlog & ReflectASM (bytecode helpers)
- FindBugs 1.3.9
- Guava (Google's Java general utils)
- ParaNamer 2.3 (Java method parameter names access)
- Netty 4.0
- Ning-compress-LZF 0.8.4 (LZF for Java)
- H2-LZF 1.0 (LinkedIn's LZF compression, used by Voldemort)
- Snappy-for-java 1.0.5
- Ganglia  (cluster monitoring)
- Metrics 3.0 (collector for Ganglia)
- GMetric4j (collector for Ganglia)
- FastUtil 6.4.4 (improved Java collections)
- Jets3t 0.7.1 (Amazon S3 Java tools)
- RemoteTea 1.0.7 (ONC/RPC over TCP and UDP for Java)
- Xmlenc 0.52 (streaming XML in Java)
- Typesafe config 0.3.1 (JVM config files tools)
- Dough Lea's concurrent (advanced util.concurrent)
- Colt 1.2.0 (math/stat/analysis lib used at Cern)
- Apache Commons, Velocity, Log4j, Slf4j