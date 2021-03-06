# gnatprove_unitstats
This Python script parses the JSON output from GNATprove runs, and computes statistics per unit/package that can be presented either in a tabular view, or in machine-readable JSON format.

## Usage

```
gnatprove_unitstats.py -t --sort=coverage,success,props --table $OBJ
```
where `$OBJ` is one path or multiple paths to `gnatprove` folders, as generated by GNATprove.
Providing multiple paths is useful for generating statistics for hierarchical projects.

Flag `-t` generates a human-readable table as follows:

```
              unit                 success    flows_su    proven      ents     coverage   coverage    props      flows   
                                               ccess                            _spec                                    
========================================================================================================================
gps                                     100        100         13         14        100        100         13          0 
servo                                   100        100         10          9        100        100         10         11 
config                                  100        100          1          1        100        100          1          0 
helper                                  100        100          0          4        100        100          0          0 
config.software                         100        100          0          1        100        100          0          0 
cpu                                     100        100          0          3        100        100          0          0 
px4io.driver                           98.4        100        124         19        100        100        126         58 
mission                                95.6        100         86         25        100        100         90         36 
main                                   94.9        100         37          4        100        100         39         37 
ublox8.driver                          89.9        100         89         28        100        100         99        135 
barometer                              81.8        100          9         11        100        100         11          1 
buzzer_manager                           75        100          9         13        100        100         12         15 
profiler                               93.3        100         14         16        100       93.8         15          4 
nvram                                   100        100         15         27        100       92.6         15         38 
logger                                 96.9        100         62         21        100       90.5         64        125 
ulog                                   98.5        100        133         15       86.7         80        135         78 
controller                             84.5        100         98         46        100       67.4        116         96 
buildinfo                               100        100         16          4        100         50         16          8 
estimator                              83.6        100        122        105        100       37.1        146        125 
led                                     100        100          0          4        100         25          0          0 
units.numerics                           50        100          7         37        100       21.6         14          0 
led_manager                             100        100          0         11        9.1        9.1          0          0 
fat_filesystem.directories.files        100        100          0         11          0          0          0          0 
fat_filesystem                          100        100          0         52          0          0          0          0 
fat_filesystem.directories              100        100          0         30          0          0          0          0 
config.tasking                          100        100          0          1          0          0          0          0 
sdlog                                   100        100          0         10        100          0          0          3 
...
```
The rows in the table are units/packages, sorted by the column names as provided with the `--sort` flag. Additionally, alphabetical order can be selected with `--sort=alpha`.

The table is followed by a summary over all units, including a classification of success by property (check) type:
```
TOTALS:
{'ent_cov': 56.26740947075209,
 'ent_cov_spec': 84.26183844011142,
 'ents': 718,
 'flows': 1064,
 'flows_proven': 1064,
 'flows_success': 100.0,
 'flows_suppressed': 14,
 'props': 1459,
 'proven': 1171,
 'rules': {u'ALIASING': {'cnt': 16, 'proven': 16},
           u'UNINITIALIZED': {'cnt': 1048, 'proven': 1048},
           u'VC_ASSERT': {'cnt': 30, 'proven': 28},
           u'VC_CEILING_PRIORITY_PROTOCOL': {'cnt': 1, 'proven': 1},
           u'VC_COMPLETE_CONTRACT_CASES': {'cnt': 1, 'proven': 1},
           u'VC_CONTRACT_CASE': {'cnt': 6, 'proven': 4},
           u'VC_DISCRIMINANT_CHECK': {'cnt': 66, 'proven': 65},
           u'VC_DISJOINT_CONTRACT_CASES': {'cnt': 2, 'proven': 2},
           u'VC_DIVISION_CHECK': {'cnt': 39, 'proven': 37},
           u'VC_FP_OVERFLOW_CHECK': {'cnt': 340, 'proven': 150},
           u'VC_INDEX_CHECK': {'cnt': 12, 'proven': 12},
           u'VC_LENGTH_CHECK': {'cnt': 109, 'proven': 102},
           u'VC_OVERFLOW_CHECK': {'cnt': 28, 'proven': 28},
           u'VC_POSTCONDITION': {'cnt': 16, 'proven': 10},
           u'VC_PRECONDITION': {'cnt': 195, 'proven': 174},
           u'VC_RANGE_CHECK': {'cnt': 613, 'proven': 556},
           u'VC_TASK_TERMINATION': {'cnt': 1, 'proven': 1}},
 'skip': 113,
 'spec': 201,
 'success': 80.26045236463331,
 'suppressed': 6,
 'units': 37}
```

## Meaning of the Columns and Totals
The following is a list of all available columns. Only a subset is shown in the table (adding/removing table columns is described below in section "Tweaking").

General:
 * **unit**: name of the package. Generics are usually not shown (depends on GNATprove version), but included at the place of their instantiation
 * **ents**: number of entities in unit
Related to properties (checks):
 * **success**: percentage of successfully proven properties in unit
 * **suppressed**: number of failed properties which were suppressed with `pragma Annotate`
 * **props**: number of properties in unit
 * **proven**: number of proven properties in unit
Related to flow analysis:
 * **flows**: number of flows in unit
 * **flows_success**: percentage of successfully proven flows in unit
 * **flows_proven**: number of successfully proven flows in unit
 * **flows_suppressed**: number of flow warnings which were suppressed with `pragma Annotate`
Related to coverage (how much of my code is in SPARK?):
 * **spec**: number of entities where spec is in SPARK for this unit
 * **body**: number of entities where body is in SPARK for this unit
 * **skip**: number of entities where neither spec nor body is in SPARK for this unit
 * **ent_cov**: percentage of entities from all units which are in SPARK (body and spec.)
 * **ent_cov_spec**: percentage of entities from all units where at least the spec is in SPARK

## Extended output in JSON mode
The following are only available in JSON output with flag `-d`, and given only per unit
 * **coverage**: number of entities where body is in SPARK divided by number of entities, as percentage
 * **coverage_spec**: number of entities where at least spec in in SPARK divided by number of entities, as percentage. coverage_spec >= coverage.
 * **rules**: counting occurences of each property type
 * **entities**: a list of all entities analyzed by GNATprove

Example:
```
[{"gps": {"body": 14, "flows_proven": 0, "success": 100.0, "rules": {"VC_RANGE_CHECK": {"cnt": 12, "proven": 12}, "VC_POSTCONDITION": {"cnt": 1, "proven": 1}}, "skip": 0, "flows_success": 100.0, "proven": 13, "flows_suppressed": 0, "entities": [{"name": "GPS", "type_orig": "K", "file": "gps.ads", "line": 8, "type": "package", "col": 9}, {"name": "GPS_Sensor", "type_orig": "K", "file": "gps.ads", "line": 16, "type": "package", "col": 12}, {"name": "initialize", "type_orig": "U", "file": "gps.ads", "line": 23, "type": "procedure", "col": 14}, {"name": "read_Measurement", "type_orig": "U", "file": "gps.ads", "line": 27, "type": "procedure", "col": 14}, {"name": "get_Position", "type_orig": "V", "file": "gps.ads", "line": 30, "type": "function", "col": 13}, {"name": "get_GPS_Fix", "type_orig": "V", "file": "gps.ads", "line": 32, "type": "function", "col": 13}, {"name": "get_Num_Sats", "type_orig": "V", "file": "gps.ads", "line": 34, "type": "function", "col": 13}, {"name": "get_Pos_Accuracy", "type_orig": "V", "file": "gps.ads", "line": 36, "type": "function", "col": 13}, {"name": "get_Speed", "type_orig": "V", "file": "gps.ads", "line": 38, "type": "function", "col": 13}, {"name": "get_Time", "type_orig": "V", "file": "gps.ads", "line": 40, "type": "function", "col": 13}, {"name": "Image", "type_orig": "V", "file": "gps.ads", "line": 44, "type": "function", "col": 13}, {"name": "Sensor_Signal", "type_orig": "K", "file": "gps.ads", "line": 26, "type": "package", "col": 12}], "ents": 14, "coverage_spec": 100.0, "coverage": 100.0, "props": 13, "flows": 0, "suppressed": 0, "spec": 0, "details_proofs": [{"severity": "info", "rule": "VC_RANGE_CHECK", "check_tree": [{"proof_attempts": {"CVC4": {"steps": 5, "result": "Valid", "time": 0.04}}, "transformations": {}}], "file": "generic_sensor.ads", "line": 29, "col": 108, "how_proved": "prover"}, {"severity": "info", "rule": "VC_RANGE_CHECK", "check_tree": [{"proof_attempts": {"CVC4": {"steps": 20, "result": "Valid", "time": 0.09}}, "transformations": {}}], "file": "gps.adb", "line": 60, "col": 47, "how_proved": "prover"}, {"severity": "info", "rule": "VC_RANGE_CHECK", "check_tree": [{"proof_attempts": {"CVC4": {"steps": 1316, "result": "Valid", "time": 0.17}}, "transformations": {}}], "file": "gps.adb", "line": 60, "col": 53, "how_proved": "prover"}, {"severity": "info", "rule": "VC_RANGE_CHECK", "check_tree": [{"proof_attempts": {"CVC4": {"steps": 1942, "result": "Valid", "time": 0.13}}, "transformations": {}}], "file": "gps.adb", "line": 60, "col": 92, "how_proved": "prover"}, {"severity": "info", "rule": "VC_RANGE_CHECK", "check_tree": [{"proof_attempts": {"CVC4": {"steps": 2576, "result": "Valid", "time": 0.22}}, "transformations": {}}], "file": "gps.adb", "line": 60, "col": 98, "how_proved": "prover"}, {"severity": "info", "rule": "VC_RANGE_CHECK", "check_tree": [{"proof_attempts": {"CVC4": {"steps": 3224, "result": "Valid", "time": 0.17}}, "transformations": {}}], "file": "gps.adb", "line": 60, "col": 137, "how_proved": "prover"}, {"severity": "info", "rule": "VC_RANGE_CHECK", "check_tree": [{"proof_attempts": {"CVC4": {"steps": 3894, "result": "Valid", "time": 0.32}}, "transformations": {}}], "file": "gps.adb", "line": 61, "col": 9, "how_proved": "prover"}, {"severity": "info", "rule": "VC_RANGE_CHECK", "check_tree": [{"proof_attempts": {"CVC4": {"steps": 4579, "result": "Valid", "time": 0.36}}, "transformations": {}}], "file": "gps.adb", "line": 61, "col": 48, "how_proved": "prover"}, {"severity": "info", "rule": "VC_RANGE_CHECK", "check_tree": [{"proof_attempts": {"CVC4": {"steps": 5285, "result": "Valid", "time": 0.34}}, "transformations": {}}], "file": "gps.adb", "line": 61, "col": 54, "how_proved": "prover"}, {"severity": "info", "rule": "VC_RANGE_CHECK", "check_tree": [{"proof_attempts": {"CVC4": {"steps": 6006, "result": "Valid", "time": 0.31}}, "transformations": {}}], "file": "gps.adb", "line": 61, "col": 93, "how_proved": "prover"}, {"severity": "info", "rule": "VC_RANGE_CHECK", "check_tree": [{"proof_attempts": {"CVC4": {"steps": 6748, "result": "Valid", "time": 0.49}}, "transformations": {}}], "file": "gps.adb", "line": 61, "col": 99, "how_proved": "prover"}, {"severity": "info", "rule": "VC_POSTCONDITION", "check_tree": [{"proof_attempts": {"CVC4": {"steps": 7384, "result": "Valid", "time": 0.56}}, "transformations": {}}], "file": "gps.ads", "line": 45, "col": 19, "how_proved": "prover"}, {"severity": "info", "rule": "VC_RANGE_CHECK", "check_tree": {}, "file": "units.ads", "line": 34, "col": 21, "how_proved": "interval"}]}},
...
```

## Command Line Flags
```
gnatprove_unitstats.py -P<gprfile>  [OPTION] (<gnatprove folder>)+

OPTIONS:
   --sort=s[,s]*
          sort statistics by criteria (s=alpha,coverage,success,props,ents,skip)
          e.g., "--sort=coverage,success" to sort by coverage, then by success
   --table, -t
          print as human-readable table instead of JSON/dict
   --exclude=s[,s]*
          exclude units which contain any of the given strings
   --include=s[,s]*
          only include units which match exactly any of given strings
   --details, -d
          keep detailed proof/flow information for each unit
```

## Tweaking
Columns in the table can be added/removed in function main:
```
...
if table:
        tablecols = ["unit","ents","success","coverage","proven","props","flows","flows_success"]      
```
Available columns as mentioned in section "Meaning of Columns".

## Installation
Requires Python 2.7 and texttable (tested with 0.8.4, https://pypi.python.org/pypi?name=texttable&:action=display). Just execute the file as explained in section "Usage".

## Known Problems
Entity count might be imprecise (see warnings) because GNATprove finds more entities than given in the ALI files.
The ALI files have to be considered to compute coverage, though.
