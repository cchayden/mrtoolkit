# Reducers #

To define your own reducer, you should write a class derived from ReduceBase.

## Declarations ##
Declarations, both field, emit, and separator declarations are the same as for mappers [Mappers](Mappers.md).

Reducer input field names are ordinarily the same as mapper output field names, but they don't have to be: you can name them anything.

## Processing ##
### Begin and End ###
`process_begin(dummy, output)`

`process_end(dummy, output)`

As with mappers, `process_begin` is called before any input records are processed.  `Process_end` is called after all input records have been processed.

### Groups ###
Reducers typically operate on groups of records sharing a common first field.  Because there are often many such records, reducers generally must operate on records one at a time: they cannot read all the records into memory and operate on them there.  To allow this style of processing, the following functions are used to process records the records with the same first field.

`process_init(input, output)`

The function `process_init` is called when a new value for the first field is encountered (or at the beginning of the input).  The initial record of the group is passed to the function.  It is also passed an output record, as are all the other process functions.  `Process_init` may return output records, although it typically does not.

`process_each(input, output)`

The function `process_each` is called for each record in a group.  The first record of the group is passed to both `process_init` and `process_each`.  Because it is usually necessary to see the whole group before doing anything, `process_each` typically returns nil.

`process_term(dummy, output)`

The function `process_term` is called after each of the members of a group have been processed by `process_each`.  By the time `process_term` is called, an input record from another group has been read, so the last record from the group is gone.  However the member variable `@last` stores the previous first field, the field that all records in the group share.  Most often, `process_term` generates all the output for the group: `@last` can be used to accomplish this.

## Output Records ##
`new_output`

As with mappers, you can get instance of output records by calling `new_output`.  This is required when you need to output more than a single record from one processing step.