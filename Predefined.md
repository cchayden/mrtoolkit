# Predefined Mappers and Reducers #

As a convenience, MRToolkit comes with a set of predefined mappers and reducers.  This collection will be expanded over time as we discover more commonly-used map and reduce operations.

## Mappers ##
### CopyMap ###
`CopyMap, copy`

CopyMap copies its input to its output.  The optional `copy` argument specifies how many fields to copy (defaults to 1).

This is useful where the purpose of the job is performed by the reduce stage, or where you want to eliminate some of the fields.

### SelectMap ###
`SelectMap, re, field, copy`

SelectMap selects records from its input based on a given regular expression.  The expression is matched against field `field`.  Field numbers are zero based. The optional `copy` argument specifies how many fields to copy (defaults to 1).

For example `SelectMap, /^GET/, 4` selects all records where field 4 (the request header) starts with GET, and outputs the first field (the IP address).

## Reducers ##

### CopyReduce ###
`CopyReduce, copy, skip`

CopyReduce copies every record from its input to its output.  The optional argument `copy` gives the number of fields to copy (defaults to 1).  The optional argument `skip` gives the number of initial fields to skip.

For example `CopyReduce, 2, 2` copies the third and fourth field of every record.

This reducer is often useful if you want to generate a field during the map phase that is used only to order the records, but which needs to be eliminated from the final output.

CopyMap and CopyReduce can be used together where the purpose of the job is primarily the sorting stage that is performed as part of the shuffle.

### UniqueReduce ###
`UniqueReduce`

UniqueReduce examines the first field of every record, and outputs only the last record of each series with the same first field.  Because the shuffle operation brings together all records with the same first field, this effectively picks one of them for output.

This reducer is useful for creating samples of the data, for each unique value of the first field.

### SumReduce ###
`SumReduce, count, skip`

SumReduce sums the values in the given fields.  The optional argument `count` gives the number of fields to sum (defaults to 1).  The optional argument `skip` gives the number of initial fields to skip.

For example, `SumReduce, 1, 2` outputs a single field, which is the sum of the values in field 3 (the first two fields are skipped).

This is useful for adding up totals across the entire input set.  In this instance the shuffle aspect of map/reduce is not being used.  For better performance, you should make sure that the first field has sufficient diversity to allow multiple reducers: if the first field of every record is identical, the system will be forced to use a single reduce stage.

Remember that each reducer produces its own output, so if you want a sum across all records, you will need to sum up the final result of each reducer separately.

### UniqueSumReduce ###
`UniqueSumReduce, count, extra`

UniqueSumReduce sums up fields within each group of records that have the same first field.  The optional argument `count` gives the number of fields to sum.
These are the next {{{count}} fields after the first.
The optional argument `extra` gives the number of additional fields to copy.
These are the next `extra` fields after the `count` fields after the first.
The values that are copied are those of the last record in the series with the same first field.  It outputs the first field, the sums, and the extras in that order.

UniqueSumReduce is the workhorse of many jobs, because it can be used to create totals within each group, where a group is defined by records with the same first field.

For example, `UniqueSumReduce` with no arguments outputs one record for each unique value of the first field.  This record contains the first field, and the sum of the values of the second field.

In another example, `UniqueSumReduce, 2, 2` outputs one record for each unique value of the first field.  It contains five fields: the unique values of the first input field, the sum within each group of the second and third fields, and the last values of the furth and fifth fields in each group.

UniqueSumReduce, like the other predefined reducers, is relatively inflexible about where things can appear in the input record.  For instance, there is no way to group by any field but the first, there is no way to sum field 4 and copy fields 2 and 5.  The grouping field must be the first: this is inherent in map/reduce.  The reason that it is acceptable to be inflexible about the count and extra fields is that you can order the fields as you wish in the mapper.  So when you need to sum some of the fields and copy others, simply write the mapper to put them in the right place in the record.

### Joint Uniqueness ###
Sometimes it is desired to group by two fields at the same time.  For instance, you might be interested in analyzing the byte count from each IP address and user agent separately.  Here is one approach: use the mapper to create a record that has the following fields:
  * IP address and user agent fields jointed by a "|" symbol
  * byte count
  * IP address
  * user agent
Then use `UniqueSumReduce 1, 2`.  This will sum up the byte counts for each unique combination of IP address and user agent, and also output the IP address and user agent separately.  A post-processing step can eliminate the first field.

`  `
### UniqueCountReduce ###
`UniqueCountReduce extra`

UniqueCountReduce counts the number of records in each group of records with the same first field.  The optional argument `extra` gives the number of additional fields to output (defaults to 0).  The extra are taken from the last record in each group.

If you simply need to identify the unique values in a field and count up the number of their occurences, use this reducer.  It is used in the examples to count the number of requests from each IP address.

### UniqueIndexedSumReduce ###
`UniqueIndexedSumReduce`

UniqueIndexedSumReduce operates on groups of records with the same first field.  It outputs one output record for each unique value of the second field.  The output record consists of the first (grouping) field, the second field, and a sum of the values of the third field (for each value of the first two fields).

For example, you can count the number of requests for each IP address for each result code by using the mapper to put the IP address in the first field, the result code in the second field, and 1 in the third field.

Because the input is not grouped on the second field, the reducer must have memory for all the values of the second field (within each group).  So this reducer is appropriate for situations where there may be large numbers of values in the first field, but within each group, only a limited number of values for the second.

This reducer provides another way to do joint uniqueness across two fields, applicable when there is a limited number of values in the second field.  The example above, using the result code for the second field, is appropriate because there are only a few result code values in use.

### UniqueIndexedCountReduce ###
`UniqueIndexedCountReduce`

UniqueIndexedCountReduce works like UniqueIndexedSumReduce, instead of summing the third field, it simply counts the number of occurrences of the different values of the second field.

### SampleReduce ###
`SampleReduce, sample_size`

SampleReduce is used to take a random sample of its input.  The `sample_size}} argument is required.  !SampleReduce looks only at the first field of each record, and outputs only this same field.  A total of {{{sample_size` output records are generated (unless there are fewer than that in the input data set).

Each record input to SampleReduce has an equal chance of being selected.  All computation is done in memory, so the sample size cannot be too large (millions).  The running time of SampleRecuce is independent of the sample size.

### MaxReduce ###
`MaxReduce, count`

MaxReduce takes groups of records with the same first field, and produces the `count` members with the highest values of the second field.

For example `MaxReduce, 3` outputs (up to) three records in each group, the ones with the three highest values of field two.

This reducer can be thought of as follows.  The input is a set of key/value pairs (the first two fields).  The value is numeric.  MaxReduce finds the record that has the greatest value for each key.

Values are compared using `<=>`. If this is not the comparison that you want, then you can override compare to perform your own comparison.  The standard compare is defined as follows:
```
  def compare(x, y)
    y <=> x
  end
```

### UniqueFirstReduce ###
`UniqueFirstReduce, copy, skip`

UniqueFirstReduce picks the first record in each group with the same first field.  The optional argument `copy` specifies how many fields to copy (default 1).  The optional `skip` specifies how many fields (after the first) to skip before copying (default 0).

For example, `UniqueFirstReduce, 1, 2` produces one record for each unique value in field 1.  The record has one field, which is taken from field three of the first record in each group.  The first (unique) field is output only if `skip` is zero (or omitted).