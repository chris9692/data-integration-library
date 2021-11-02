# extract.table.name

**Tags**: 
[gobblin](categories.md#gobblin-properties)

**Type**: string

**Default value**: no default, required, blank value will abort the job

**Related**:

## Description

`extract.table.name` specifies the target table name, not the source table name. This
is a required parameter. Writers and some converters don't work without it. The job
will abort if this property is blank in job configuration.


[back to summary](summary.md#essential-gobblin-core-properties)
