# USING_PERSONAL_WINDOWS_LAPTOP_MACHINE_TO_RUN_TOPOLOGY_APACHE_STORM

A Linux-based Virtual environment can be created in a Windows-based personal laptop machine and real-time streaming framework can be set up. The framework will be a combination of – single node cluster, Apache Zookeeper, Apache Storm, Ubuntu, Eclipse IDE, JAVA, Python, Apache Maven and Oracle Virtual-Box. The article will act as a suitable guide to create this combination, build a real Storm topology, run the topology, and analyze results/logs. The scope of the article will not include step-by-step guide to create virtual system set up but just an overview of it. Details of those steps can be found via Internet sources. The major focus of this article will shift towards the details of the Storm Topology that will be used to run in the machine.

An Apache Storm topology that counts the words in a file will be written and run using the virtual environment. The topology was referenced from a published book “Getting Started With Storm” by Jonathan Leibiusky, Gabriel Eisbruck and Dario Simonassi (O’Reilly).” As per the book’s advice, I will not copy-paste the entire code here in this article since I don’t have permission to do it. To get the entire code, please purchase the book. However, I will discuss key sections of the topology in relevance to Apache Storm real-time streaming framework and API and cover topics like ‘guaranteed message processing’, ‘parallel-processing’, ‘running topology in local-cluster’, ‘configurations’ etc. I will then finally display the command to run the topology in Ubuntu machine terminal and also display results and discuss the logs. 

## Setting Up Virtual Machine

The first step to create a virtual environment to execute Storm Topology is to install a virtual machine. An Oracle VM VirtualBox was used for this purpose. This virtual machine can be downloaded free from internet-based source and installation process is also simple and straightforward.

The second step is to set up Ubuntu. Ubuntu can be downloaded from http://www.ubuntu.com/download. Out of the various options to download, a CD image was selected to be downloaded.

The starting point to set up Ubuntu is to run the virtual machine and click on ‘New’. (see VMsetup.png)

A new virtual machine with Ubuntu was created following series of manual steps. The individual steps will not be documented in this article as it doesn’t serve the purpose of the article. Step by step guide to set up is easily available via Internet and is easy.


## Setting Up Apache Storm

The third step after Ubuntu set up is to install Apache Storm and Apache Zookeeper in the Ubuntu system.
Following is a list of steps to perform:
	Check for the system requirement that includes JAVA version and Python (atleast 2.6).
	Installation should begin with Zookeeper installation:
	Download Zookeeper set up from Apache Zookeeper site.
	Untar the Zookeeper installation folder.
	Within the ‘conf’ folder, locate the file ‘zoo.cfg’ and update the file with relevant values for: tickTime, dataDir, clientPort.
	The next step is to install Storm:
	Fetch the storm installation folder from Apache site and unzip it.
	Locate the ‘storm.yaml’ file and update information relevant to:
        storm.zookeeper.servers, storm.zookeeper.port, nimbus.host, storm.local.dir, java.library.path, supervisor.slots.ports,   
	worker.childopts, nimbus.childopts, supervisor.childopts
	Start the Zookeeper server with the command zkServer.sh start
	Start the nimbus daemon with the command bin/storm nimbus
	Start the storm ui with bin/storm ui
	Start the storm supervisor with bin/storm supervisor

These are the key steps to get things set up. However, the details of steps are not included. For this, I suggest to refer to storm installation information available in various Internet sources. The scope of this article doesn’t cover comprehensive explanation of all the steps needed for environment and system set up. However, explanation of Storm Topology, execution of topology and result analysis will be handled with details.

## About Our Topology:

The Storm topology that will be used during the experiment will be the ‘Word Count’ topology that counts words present in an input file. The topology will consist of three components:

#### WordReader Spout
#### WordNormalizer Bolt
#### WordCounter Bolt

Before we cover each code in details, let’s talk about how to build the project once we finish writing the application source codes and putting them in folder just like for any JAVA application. The purpose here is to import Storm dependencies (a set of jars to include in application classpath). For this there are two options:
a.	Download the dependencies, unpack them, and add them to classpath.
b.	Use Apache Maven.
For option ‘a’, simply using IDE like Eclipse and importing necessary jars would suffice.
For option ‘b’, a pom.xml file needs to be written and included in the application folder. The scope of this article doesn’t include details about Apache Maven. However, pom.xml file (see mavencode.png) content needed for our project. It shows storm dependency needed for the project:

Now, the straight focus of this article is on the Spout – ‘WordReader’. As I already stated in the ‘Abstract’ section of this article that I don’t have the permission to copy paste the entire code, hence, I will pick key sections of the spout source code and discuss the details surrounding it.
The first part of the code that I pick is the class description which is represented in the following line of code:

public class WordReader extends BaseRichSpout {} 

As seen above, our spout is a subclass of another class – BaseRichSpout. BaseRichSpout is an abstract class designed as a template. It has the following signature:

public abstract class BaseRichSpout extends BaseComponent implements IRichSpout

BaseComponent is an abstract subclass of Object class and implements an interface ‘IComponent’. Following are the two methods under IComponent:

Map<String, Object> getComponentConfiguration()

With this method, a component-specific configuration can be set up. The configuration specified as a Map type will be limited to the spout only.

void declareOutputFields(OutputFieldsDeclarer declarer)

With this method, the values emitted by the spout component can be given field/fields name(s). Our WordCount topology declares the output field as ‘line’.

IRichSpout is an interface that is implemented by BaseRichSpout. Technically, all the spouts in Storm directly or indirectly implement this interface. IRichSpout follows JAVA multiple inheritance pattern and inherits from two other interfaces – ISpout and IComponent. Following is the list of methods owned by IRichSpout:

(i)	void open(Map conf,TopologyContext context,SpoutOutputCollector collector)

This method is called as soon as an individual task for the spout is instantiated within the memory of a worker process on the cluster. ‘conf’ holds Storm configuration for the spout, ‘context’ carries information about this task in the topology and ‘collector’ is used to emit data from the spout. ‘open_method.txt’ shows the implementation of same for our WordCount topology:

Hence, for every instantiation of our WordReader spout in the memory within each individual worker process across the cluster, a ‘filereader’ object is instantiated. The local variable ‘collector’ is passed to the instance variable with the same name in the class.

(ii)	void close()

called when ISpout is going to be shut down. There’s no guarantee that this method will be called. However, this method is guaranteed to be called during local cluster mode when the topology is killed.

(iii)	void activate()

called when spout is activated out of deactivated mode. nextTuple() method will be called soon. The relevant scenario here is when using storm-client to deactivate and activate topology.

(iv)	void deactivate()

called when spout has been deactivated. nextTuple() will not be called until activation.

(v)	void nextTuple()

with this method on top of the stack, storm requests spout to emit tuples to the output collector. ‘nextTuple_method.txt’ contains the implementation of this method within our topology – WordCount. The method emits every new line from the input file.
     
First time this method gets on top of stack, ‘completed’ is set to default value ‘false’. The ‘if’ statement is skipped and execution moves to the next part. The ‘reader’ then reads lines from the file as long as new lines exist and the lines are emitted via ‘collector’. Once, no more lines are present to be emitted, ‘completed’ is set to ‘true’. Since nextTuple() method runs forever, the execution returns to caller and now ‘if’ statement is run. Thread.sleep(1000) method is then called to pause the method to reduce CPU load. After every 1000 milliseconds of pause nextTuple() tries to execute and goes back to rest since ‘completed’ is still ‘true’. This happens until Topology is killed/shutdown.

Hence, sleeping in nextTuple() is critical to reduce CPU overload. However, as per Nathan Martz (as shown in the snippet ‘nmarzstatement.png’), we don’t need to sleep in nextTuple() anymore.

Instead, it can be configured via the spout wait strategy config.
Finally, the above method emits a single line from the file as a message with message identifier as the same message.

(iv) void ack(Object msgId)

called when a tuple with the message identifier has been fully processed by Storm. The message is acknowledged and taken off the queue. Our WordCount topology overrides the method like:

public void ack(Object msgId) {
		System.out.println("OK:"+msgId);
	}

Hence, for every line emitted, storm prints out a line containing the message identifier.

(vi)	void fail(Object msgId)

called when a tuple with the message identifier fails to be fully processed by Storm. The message is put back in the queue for replay. Our WordCount topology overrides the method in following way:

public void fail(Object msgId) {
		System.out.println("FAIL:"+msgId);
	}

## Final Note:
Other than the methods inherited from java.lang.Object that include (clone(), equals(), finalize(), getClass(), hashCode(), notify(), notifyAll(), toString(), wait(), wait(), wait()), our spout ‘WordReader’ own the methods namely – declareOutputFields(), getComponentConfiguration(), open(), nextTuple(), ack(), fail(), activate(), deactivate(), close().

Now, we will move towards our first bolt – WordNormalizer:
The first part of the code I am going to pick to discuss is:

public class WordNormalizer extends BaseBasicBolt {}

So our bolt – ‘WordNormalizer’ is a subclass of the class ‘BaseBasicBolt’ that has the following signature:

public abstract class BaseBasicBolt extends BaseComponent implements IBasicBolt

‘BaseComponent’ has the following class definition:

public abstract class BaseComponent extends Object implements IComponent

Hence, just like our spout – ‘WordReader’, our bolt also inherits the methods owned by the class ‘BaseComponent’

Map<String, Object> getComponentConfiguration()

void declareOutputFields(OutputFieldsDeclarer declarer)

Interface ‘IBasicBolt’ owns three additional methods which are:-

void prepare(Map stormConf,TopologyContext context)
void cleanup()

called only in local cluster mode when topology is shut down.

void execute(Tuple input,BasicOutputCollector collector)
 
called to process the input tuples and optionally emit new tuples. Acking is auto-managed. ‘execute_method.txt’ contains implementation of this method in our bolt – ‘WordNormalizer’ –

Hence, the bolt takes input as a line emitted by the spout and emits all words in the line.

The second and final bolt in our topology is the ‘Word Counter’. Key implementations include ‘prepare_method.txt’ with which we will create some resources during the instantiation of our bolt.
Using ‘execute_word_counter_method.txt’, we will count the incoming words and create a key value pair with ‘word’ and the corresponding ‘count’.
‘cleanup_method.txt’ will print the words and the counts after the topology has been shut down.
The final piece of code to discuss is the TopologyBuilder class for our topology. The key portion of the code will look like ‘topology.txt’.
  
## Running the Topology

We now need to open the terminal in Ubuntu desktop and enter the Storm installation directory that has folder named ‘bin’. The following command then needs to be run:

bin/storm [space]  jar [space]   /path/name of storm jar [space] /package/name of Topology class [space] /path/input file

‘jar’ will transport the jar to Nimbus. ‘package’ refers to the name of the package that holds our main topology class. The argument to be supplied is the input filepath.

When we hit enter, storm starts streaming using configurations from default.yaml, storm.yaml and other component-specific configurations.

### Good Luck.

## Analyzing Results

‘finallog.txt’ is the log file after successful run.

‘ack_execution.png’ represents execution of the method ‘ack’ as seen in the log within our spout class.

‘wc_result.png’ represents execution of ‘clean’ method as seen in the log within our second bolt class.

## End Notes

During this experiment, I used Eclipse IDE, imported storm jars into classpath, created storm application jar and executed.

I used the input file with content as seen in ‘input.png’.

I suggest readers to experiment the project by themselves with any input file.

Also, I wish everyone LUCK on streaming real-time using Apache Storm.
