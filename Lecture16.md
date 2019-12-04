# MapReduce

## Algorithm

- mapper function converts single rows in the old table into multiple rows in the intermediate table
- reducer function aggregates data associated with each distinct key in intermediate table
  - makes each key a single row in the final table