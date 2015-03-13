# Examples #

The `examples` directory contains some samples jobs.  These were designed to illustrate various constructs and techniques.

## ip.rb ##
See [source](http://code.google.com/p/mrtoolkit/source/browse/trunk/examples/ip.rb).

This is the simplest example, containing only a job class.  It uses the predefined CopyMap, which simply copies its input to the output with no processing.  The predefined reducer UniqueCountReduce counts the number of each unique instance of the first field, and outputs the field and its count.

Hadoop, and MRToolkit, leaves the reducer output in HDFS, in as many files as there were reduce jobs.  Because it is not particularly useful in this form, a subsequent processing step ordinarily copies all the reduce output files to the local file system, then concatenates and sorts them.  These steps are done in the included Rakefile tasks.

In this case we want to sort on the second field (the count) so we see the IP addresses that generated the most hits.

## section.rb ##
See [source](http://code.google.com/p/mrtoolkit/source/browse/trunk/examples/section.rb).

This job extracts the section name (the first directory segment of the requested path) and counts the occurrences of each one.

The mapper process step extracts the section name and adds a count (1).  The predefined reducer UniqueSumReduce sums these up.  This example could have omitted the count and used UniqueCountReduce, but was done this way to introduce the sum reducer.

## ip-size.rb ##
See [source](http://code.google.com/p/mrtoolkit/source/browse/trunk/examples/ip-size.rb).

This job adds up the size of all the results returned to each distinct IP address.

It illustrates how to define input and output fields.  Its process step picks up selected input fields and places them in the output.  This is pretty typical: the mapper is often used to select and rearrange the input fields.  Remember that the output of the mapper is shuffled, so that all records with the same first field are fed to the (same) reducer together.

Here UniqueSumReduce actually has something to add up (rather than just counting as in the last example).

## hour.rb ##
See [source](http://code.google.com/p/mrtoolkit/source/browse/trunk/examples/hour.rb).

This job summaries the traffic by hour.  It counts the requests that occur in each hour, across all the days.

This illustrates an atypical mapper.  A more straightforward implementation would have extracted and output just the hour.  Then UniqueCountReduce could have been used to count up each of the numbers (0-23).

In the implementation given, however, the mapper creates an array of 24 counters, and it's process function increments one of these counters for each input record.  The function returns nil, so no output is produced yet.  At the end, these arrays are written out.

So each mapper writes only 24 lines of output.  The reducer needs only sum these up to get the final result, which it again uses UniqueSumReduce to do.

The functions process\_begin and process\_end are used to (1) initialize the counter array, and (2) write the mapper output.  You should only define these functions if you need them.

## top-file.rb ##
See [source](http://code.google.com/p/mrtoolkit/source/browse/trunk/examples/top-file.rb).

This job finds the top 10 most frequently requested files (using the full request path).

The map stage by this time should be familiar: it simply picks out the path and adds a count of 1.

The reducer, MaxUniqueSumReduce, is a version of UniqueSumReduce which only retains a given number of values (given by a parameter, in this case 10).  By retaining only the largest counts, the output is substantially reduced.

Because Hadoop may use many reducers, and each operates independently, each will output a different 10 highest file-count pairs.  These are not guaranteed to be the highest overall, since each reducer sees different pairs.  But the output _is_ guaranteed to have to 10 highest counts overall.

## ip-ua.rb ##
See [source](http://code.google.com/p/mrtoolkit/source/browse/trunk/examples/ip-ua.rb).

This job counts the number of distinct (IP address, user agent) pairs.  Its mapper illustrates the technique of building a synthetic field that combines the values of two input fields, for the purpose of counting.  For simplicity, the user agent is shortened to its first token. The individual fields are also output, so that a post processing step (in the Rakefile) can trim off the synthesized field while leaving behind the original fields.

## ip-result.rb ##
See [source](http://code.google.com/p/mrtoolkit/source/browse/trunk/examples/ip-result.rb).

This job counts the number of distinct (IP address, result code) pairs.
Because there are only a limited number of distinct result codes, the UniqueIndexedCountReduce reducer can be used.  For each value of the first field, it counts the number of times each different value in the second field appears.  It outputs the first and second field and the count, for each unique pair.

An alternative implementation would have been to have the mapper output a third field, containing the integer 1.  Then the reducer would have been UniqueIndexedCountReduce.
Another context where UniqueIndexedCountReduce would make sense is to sum up values in the input.  For instance it could be used to total the number of bytes returned for each combination of IP address and user agent.  To do this, the mapper would output the IP address, the user agent, and the byte count.

## import.rb ##
See [import.rb](http://code.google.com/p/mrtoolkit/source/browse/trunk/examples/import.rb)
and [import-logs](http://code.google.com/p/mrtoolkit/source/browse/trunk/examples/import-logs).

This is not a Hadoop job, but an import program for Apache log files.
It takes a raw file on standard input and produces a clean up and delimited one on standard output.  The shell program `import-logs` creates a directory in HDFS, feeds each of the raw log files into import.rb, and stores the result in HDFS.

The import program simply uses a regular expression to parse the log file.