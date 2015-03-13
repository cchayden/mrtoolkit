# Mappers #

To define your own mapper, you should write a class derived from MapBase.

The mapper defines two things: the field names of the input and output records, and the processing function that produces an output record from an input record.

## Declarations ##
### Field Declarations ###
The field names are defined in a function `declare` that you should override.
Declarations use the `field` statement (for the input) and the `emit` statement (for the output).

For example, the following declares the fields of the Apache combined log format.  The output record consists of two fields, `ip` and `size`.
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

    emit :ip
    emit :size
  end
```

Records are delimited by newline characters; fields within a record are delimited by TABs, although this can be changed (see below).  The framework reads records, parses them into fields, which are essentially ruby `Struct` objects.  So the fields of the input record can be referred to in the following ways:
  * `input.ip` The "ip" (first) field of the input
  * `input.dt_tm` The "dt\_tm" (fourth) field of the input
  * `input[0]` The "ip" (first) field

### Separator Declarations ###
You can define field separators with `field_separator` and `emit_separator` statements.  For example:
```
  def declare
    ... field and emit statements ...

    field_separator "|"
    emit_separator ","
  end
```
If possible, it is easiest to convert to the standard (TAB) field separators in the import step.

### Executable field and emit ###
Field and emit are designed to look like declarations, but in reality they are executable statements.  They are class methods of MapBase.  It is a standard feature of ruby that they can be "called" without using parentheses.

Normally you do not need to think about this.  But it does allow you to make these "declarations" dynamic.  For instance, you can define three fields this way:
```
    (0..2).each {|i| field "col#{i}"}
```
The fields are "col1", "col2", and "col3".  This example also illustrates that the argument to "field" can be a string as well as a symbol.

This facility is used to define generic mappers, such as CopyMap.  It could also be used to write a generic mapper that reads its field names from a file or from an array of strings passed into the constructor, as follows:
```
  @fields.each {|f| field f}
```

## Processing ##
### Processing Functions ###
Each input record is processed by the "process" function.  But first the mapper is given a chance to initialize itself.  After all records have been processed, the mapper is given a chance to clean up.

Each of the process functions accepts two arguments: an input record and an output record.  In some of the cases the input records are not defined, and in this case a dummy argument of nil is passed instead.  The output record is for convenience: if the processing step wants to return output it need not instantiate one, it can simply return the one passed to it.

The return value of each of these functions is either:
  * nil  - No output record is generated
  * output - A single output record passed to the reducer
  * Array of output - Each of the output records is passed to the reducer

#### process\_begin(dummy, output) ####
`Process_begin` is called before any input records.  It is normally used to initialize member variables, so it ignores its arguments and returns nil.

#### process(input, output) ####
`Process` is called on each input record.  `Input` is a `Struct` with input fields as described above.  The mapper generally produces a single output record by filling in `output`, or else produces no output by returning nil.

`Process` can filter its input, reorder the fields, clean up and reformat the field values, and generally prepare for the next steps.  Less often, `process` produces new fields that are built from values in the input.  This is done, for instance, to prepare for joint uniqueness [See predefined reducers](Predefined.md).

#### process\_end(dummy, output) ####
`Process_end` is called after all input records have been processed.  It can clean up and release resources.

The normal case in which `process_end` is used is when the mapper is doing some kind of accumulation.  In this case, `process` records some information in member variables and returns nil.  Then `process_end` is used to produce all the output.  In this case it would be returning an array of output records.  The next section shows where it gets these.

### Output Records ###
#### new\_output ####
If any of the processing functions needs to return multiple output records, it can do so by returning an array.  It needs instances of the output record class to do so.  These records can be produced by calling the function `new_output`.

### Copy Function ###
#### copy\_struct(src, dest, skip = 0) ####
The function `copy_struct` is implemented in the base class, and is useful for copying records from input to output without having to know the field names.  Because input and output types can be references by numerical index as well as by field name, you can make your own generic processing functions.  But for the simplest cases, this one will do.

`Copy_struct` copies fields from the source structure to the dest structure by position.  If specified, the initial `skip` fields in the source are skipped.  All remaining fields in the source are copied.  Note that the _field names_ are not copied, only the values.  The field names are determined by the field and emit declarations.