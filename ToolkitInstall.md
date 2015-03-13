# Installation #

## Installing MRToolkit ##

MRToolkit requires no installation other than downloading and unpacking.

If you have downloaded the tar file, unpack by:
```
  tar xfz mrtoolkit.tar.gz
```

## Configuring MRToolkit ##
Life will be simpler if you specify where to find mrtoolkit.  For example, you would include the following in `.bash_profile`:
```
export RUBYLIB=$HOME/mrtoolkit/lib
```
If you don't want to do this, you can accomplish the same thing in a Rakefile.  For example, see `examples/Rakefile`.

## Running Toolkit Tests ##
The `test` directory has unit tests.  You can run these by:
```
  cd $HOME/mrtoolkit/test
  rake test
```

## Installing Hadoop ##

Normally, you install and run MRToolkit on a system that already has Hadoop.  So that will not be described in detail here.  For complete instructions, see [Cluster Setup](http://hadoop.apache.org/core/docs/current/cluster_setup.html).

But you might want to install a simple non-cluster version of Hadoop to test out your scripts on a data subset, to get them working.  See the Hadoop [Quick Start](http://hadoop.apache.org/core/docs/current/quickstart.html) for instructions.

Note: At the time of this writing, the latest STABLE version of Hadoop was still 0.18.3. You are encouraged to use the latest version though (0.20.x). Cluster Setup and Quick Start instructions assume you are using Hadoop 0.20.x on newer.

If you follow the Quick Start instructions carefully, you will get a working version of Hadoop.  One more thing you must do is to set and export `HADOOP_HOME` and put this in your path.  You should also define `HADOOP_STREAMING_VERSION`, particularly if it is different from what appears below.  You can determine the proper version by looking at
`$HADOOP_HOME/contrib/streaming/hadoop-<version>-streaming.jar`.
```
export HADOOP_HOME=$HOME/hadoop-0.20.0
export HADOOP_STREAMING_VERSION=0.20.0
PATH=$PATH:$HADOOP_HOME/bin
```
Finally, you need to create a home directory for yourself in Hadoop File System:
```
hadoop fs -mkdir /user/<your login name>
```

## Using the Standalone Environment ##

In Hadoop, your job runs on computers in the cluster, ordinarily not the one you are running your program on.  This presents obstacles to debugging, if something goes wrong.  Even if you run in the Quick Start environment locally, the jobs execute as part of Hadoop, and it can be hard to see what they are doing.

To make it simpler to debug, a **standalone** environment is provided.  It simply concatenates all input files, pipes them to the mapper, sorts the output, pipes the result to the reducer, and stores the output.  It also simulates the Hadoop File System.

To use this standalone environment, just include `mrtoolkit/standalone` in the path before `$HADOOP_HOME` as follows:
```
PATH=$HOME/mrtoolkit/standalone:$PATH
```

Now you can write trace statements such as
```
STDERR.puts "trace message"
```
in the mapper or reducer, and see the result on the console.  Remember, whether it is the standalone environment or actual Hadoop, `STDOUT` is reserved for the data stream, so you cannot use it for trace output.

The standalone environment redefines the `hadoop` command, but only a small subset of its capabilities.  It will do:
  * hadoop fs -rmr
  * hadoop fs -rm
  * hadoop fs -mkdir
  * hadoop fs -put
  * hadoop fs -cat
  * hadoop fs -ls
  * hadoop fs -lsr
  * hadoop jar

The standalone Hadoop File System is simulated in the current directory. So if you have an import step, be sure to run it in the standalone environment, and make sure that you put the imported files somewhere different from their original place.

Now you can try the included examples. To make sure you run these in a standalone environment, run `which hadoop` and make sure it returns `mrtoolkit/sim/hadoop`.

## Running Examples ##
To try the example import, change current directory to `mrtoolkit/examples` and run
```
rake import
```

This command imports raw Apache logs included in `mrtoolkit/sample-data/raw-logs` and creates a `logs` directory with the imported files. For more details on the import job, see [import.rb](http://code.google.com/p/mrtoolkit/source/browse/trunk/examples/import.rb)
and [import-logs](http://code.google.com/p/mrtoolkit/source/browse/trunk/examples/import-logs).

Now you can try various example jobs included with MRToolkit. For example, let's count the number of unique IP addresses (first field) in the log files, and output each address and its count:
```
rake ips
```

To see the result run
```
more ips
```

You can see other example jobs included with MRToolkit by going to `mrtoolkit/examples` and running
```
rake -T
```

For more details on this and other example jobs, see [Examples](http://code.google.com/p/mrtoolkit/wiki/Examples)