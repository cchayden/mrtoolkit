# Jobs #

Every MRToolkit program must define a job class: a class that derives from JobBase.  The job class defines a mapper class, a reducer class, one or more input directories, and an output directory.

The statements in the job class are, in reality, class methods of JobBase, and are typically used without parentheses so that they resemble declarations.

## Stages ##

Jobs are composed from a series of stages, each of which contains a mapper and a reducer.  Each stage is defined by a separate method, named "stage1", "stage2", etc.  Stages are executed in numerical order.  Each stage takes files from one directory and produces files in a different one.  These can be chained together in a mutli-stage program.

If the job consists of a single stage, the function can be named "job".  If you define "job" then you should not define stage methods (although if you do, the job will be run first, then the stages).

## Mapper ##

`mapper mapper_class, args`

The mapper statement gives the name of the mapper class and any required or optional arguments it may take.  Each mapper class defines its own argument conventions.

The framework instantiates one instance of a mapper class (for each Hadoop mapper job) and feeds it records from the input file(s).  Mapper classes should derive from MapBase.

## Reducer ##

`reducer reducer_class, args`

The reducer statement gives the name of the reducer class, and any required or oe=ptional arguments.

The framework instantiates one instance of a reducer class (for each Hadoop reducer job) and feeds it records from the shuffled output of the mappers.  Reducer classes should derive from ReduceBase.  The output of each reducer job is written into a separate file in the output directory.

## Input and Output Directories ##

`indir input_dir`

The `indir` statement gives the input directory.  All files in this directory are processed by the mapper.  The argument should be a pathname (absolute or relative) within HDFS.  Relative pathnames are relative to your HDFS home, typically `/user/<your login>`.

You can specify as many `indir` statements as you need.

`outdir output_dir`

The {{{outdir}} statement gives the output directory.  This is a directory within HDFS where the reducer output files are to be written.  This may or may not exist: if necessary the directory is created or cleared out before beginning.

Be careful that you do not have something of value stored in the output directory.  Hadoop refuses to run if the output directory already exists, because it wants to protect against deleting or overwring valuable data, which may have been very costly to compute.  Because it is inconvenient to do manually, MRToolkit removes this restriction, but you should be careful.

## Options ##

`reducers n`

By default, the job uses a single reducer.  This works best if the number of records fed to the reducer is small.  If the reduce phase takes a long time, it might be well to increase the reducer count.

`extra file`

Map and reduce operations take place on different servers.  Bor them to execute, the code has to be packaged up and sent to each of the servers.  The framework handles this for the files it knows about.  But if your program requires other files, either source files included or required, or data files that are read at runtime, it must let MRTools know about them.  This statement names an "extra" file that is copied to the destination system.  These files may be anywhere (may be named by absolute or relative pathname) but are placed in the current execution directory when map or reduce is run.

`map_opt n, v`

{{reduce\_opt n, v}}}

Hadoop provides a way to pass options to the map and reduce programs.  Options are name/value pairs that are passed to the "jobconf" parameter of Hadoop.  You may define as many of these as you need.

## Running The Job ##
If you name the job class so that it ends in "Job" (for example, MainJob) then !MRToolkit will run it automatically.  If you choose to pick a different name, then you need to call the "run\_command" class method.  For example:
```
  class MyJobClass
    ...
  end
  MyJobClass.run_command
```
If you name the class according to the recommended convention, you should **not** explicitly call run\_command.