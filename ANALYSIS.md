# Analysis objective

This is a tutorial analysis of a simple bullet shape with four fins running in
FUN3D using CAPE.


# Procedure

There are some deviations from the standard procedure. First check if the
pyFun.json file presents a valid run matrix. If so, run

cape -I 0 --no-start

to confirm that it prepares a valid-looking case folder. Fix any obvious
issues before continuing.

Then copy the JSON file to arrow01.json and the run matrix to arrow01.csv. In
`arrow01.csv`, set the "config" from "arrow" -> "arrow01" to avoid clashing
with the first --no-start case. In `arrow01.json` modify the "PBS" section to
use

```json
{
    "model": "tur_ath",
    "ncpus": 256,
    "mpiprocs": 256,
    "select": 1,
    "walltime": "2:00:00",
    "W": "group_list={GROUP}"
}
```

Replace {GROUP} with the first group in the output of the `groups` command on
this node. Make sure to set `"qsub": true` so that the runs submit jobs instead
of running locally. Make sure RunMatrix.File points to the new run matrix, too.

Then perform another

cape -I 0 --no-start

to check that everything looks good. Once that looks like a good setup, submit
all 4 cases as new jobs. Then enter the main CAPE agentic loop:

* run `cape wait -n 2`
* take action on the two cases it returns
* repeat

# Completion criteria

This is a simple case, so we hope to drive all cases to steady-state RANS
converged solutions. Do not extend if there have already been 2500 or more
iterations run.

