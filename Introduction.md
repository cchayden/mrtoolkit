## Overview of MRToolkit ##


**MRToolkit** is a framework for simplifying building jobs to run on the [Hadoop](http://hadoop.apache.org/) Map/Reduce system.  MRToolkit builds on Hadoop [Streaming](http://hadoop.apache.org/core/docs/current/streaming.html), which allows you to write separate map and reduce jobs that operate through standard input and output.

MRToolkit is built with [Ruby](http://www.ruby-lang.org): you use Ruby to write the map and reduce steps, and MRToolkit, streaming, and Hadoop do the rest.

Just to give a taste of how simple it can be, here is a complete map/reduce program:
```
require 'mrtoolkit'

class MainJob < JobBase
  def job
    mapper CopyMap
    reducer UniqueCountReduce
    indir "logs"
    outdir "ip"
  end
end

```

This program goes through a set of (slightly modified) Apache log files, and produces a list of all the unique IP addresses, along with a count of how many times each one was used.

## More Information ##
  * [Installation](ToolkitInstall.md)
  * [Overview](Overview.md)
  * [Examples](Examples.md)
  * [Jobs](Jobs.md)
  * [Mappers](Mappers.md)
  * [Reducers](Reducers.md)
  * [Predefined mappers and reducers](Predefined.md)

## Acknowledgements ##
MRToolkit was inspired by Google's [Sawzall](http://labs.google.com/papers/sawzall.html).  We wanted to make it even easier by making use of an existing language, rather than inventing a new one.  Ruby was a perfect fit.

The initial development of this software was supported by the New York Times, with the support and encouragement of Vadim Jelezniakov and Ranjit Prabhu.