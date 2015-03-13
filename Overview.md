# Overview #

MRToolkit constructs a **job** and submits it to Hadoop for execution.  The job consists of two parts: a map program and a reduce program.  Each of these may be executed (on different parts of the input data set) on different servers.

In a regular **streaming** job, each of these two program gets input from standard input, parses each line into records, processes these records, constructs an output record, and sends it to standard output.

With MRToolkit, most of this is handled by the framework.  You have to
  1. Define a (mapper or reducer) class that derives from a framework (mapper or reducer) base class.
  1. Declare the names of the input and output fields.
  1. Define the processing of individual records.
  1. Declare the location of the input files, and where to put the output files.

To make things even simpler, MRToolkit comes with a set of predefined mappers and reducers, so you might not have to even write a mapper or reducer class.

## Details ##
In practice, you usually wind up writing a mapper class, but can often make use of a standard reducer.  Mappers are simpler to write, because they ordinarily process one record at a time and retain no memory from one record to the next.  Mapper state is impractical, because you cannot be sure whether two records will even be processed by the same mapper instance.

Reducers ususally _do_ have state, because the whole point of reducers is to process groups of related records that have been brought together by the shuffle.  This makes them harder to write.  See how the predefined reducers are implemented for ideas on how best to do this.

## Mapper ##
The mapper usually consists of two parts: input and output field declarations and a processing step.

Input field are declared in a `declare` function, using the `field` tag.
Output fields are declared using an `emit` tag.  Here is an example, which describes the Apache Combined Log format, and which outputs two fields "path" and "count".
```
  def declare
    # declare log fields
    field :ip
    field :client_id
    field :user_id
    field :dt_tm
    field :request
    field :status
    field :result_size
    field :referer
    field :ua

    emit :path
    emit :count
  end
```

In addition to the field declarations, the mapper transforms each input record into an output record.  This is normally done by defining `process`.

In the following example, the processing function looks at the HTTP request header, and extracts the first segment of the pathname of GET requests.
```
  def process(input, output)
    if input.request =~ /GET\s+(\S+)\s/
      output.path = $1
      output.count = 1
    end
    output
  end
```
The function is passed an input record, populated with fields corresponding to the declaration.  It is also passed en empty output record (so that the routine does not have to instantiate one).  We see here that if the request is GET, then it sets the output record to the pathname of the file requested.

The function normally returns the output record, which is eventually passed to the reducer.  If the function returns `nil` then this input record does not generate output to the reducer.

See [Mappers](Mappers.md) for more detail on building mappers.

## Reducer ##
Since you rarely have to build reducers, they will not be discussed further here.  See [Reducers](Reducers.md) for more details.

## Job ##
The job class declares how to assemble the Hadoop job.  It defines the mapper and reducer class names, the input directory, and the output directory.

For example, here is an example:
```
class MainJob < JobBase
  def job
    mapper MainMap
    reducer MaxUniqueSumReduce, 10
    indir "logs"
    outdir "top-file"
  end
end
```
All job specifications are in a `job` function.
The mapper and reducer declarations speficy the names of the mapper and reducer classes.  In this case, MainMap is a custom mapper, and MaxUniqueSumReduce is a predefined class that takes one parameter (10).
The indir declaration specifies the input directory (in HDFS): all the files in this directory will be processed.
The outdir declaration specifies the output directory (IN HDFS): it will be cleared out if it exists, otherwise it will be created.

See [Jobs](Jobs.md) for more details.

### Notes ###
The Combined Log Format is described [here](http://httpd.apache.org/docs/1.3/logs.html).

Streaming, thus MRToolkit, requires field delimiters.  Because the log files as produced by Apache do not provide such delimiters, a log file importer rewrites the files as it copies them to the Hadoop File System, where they must reside to be accessible to Hadoop.

The supplied importer (found in `examples/import.rb`) splits the log files, removes delimiters such as the brackets around the time and the quotes around several other fields, and inserts TABs as field delimiters.