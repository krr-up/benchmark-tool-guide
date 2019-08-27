## Running vanilla clingo
This part of the guide is dedicated to showing how to modify the [example runscript](https://github.com/potassco/benchmark-tool/blob/master/runscripts/runscript-example.xml) so that it can run clingo with its default settings on a SLURM cluster. Hopefully, while reading this you learn which parts you need to modify to run your clingo variants/programs.
The modified runscript can be found in the files folder.

## Changes to the runscript
Most changes to the runscript are very simple. The first change is on line 1. We change the value of *output* to "vanilla-clingo" since we want our benchmark folder to have a meaningful name.

The second change is on line 7. Here, we want to define which script to use to run clingo. There is no default script we can choose that just runs regular command line clingo so we will make one ourselves. First, we want to change the value of *version* to "vanilla". The file that will be called when running the benchmarks is named "clingo-vanilla" and has to be in the benchmark-tool/programs folder. Assuming that clingo is installed on your system the file is very simple:
```
#!/bin/bash
clingo $@
```
After writing this file, remember to make it executable. This file can be found in the files folder.

#### Multiple settings
Let's say that for our experiments we want to see how the "jumpy" configuration compared to the base clingo configuration. So, we define two settings:
```
<setting name="base" cmdline="'--stats --quiet=1,0'" tag="basic" />
<setting name="jumpy" cmdline="'--configuration=jumpy --stats --quiet=1,0'" tag="jumpy" />
```
Notice the changes to the values of *name* and *tag* so that they are descriptive. Also, see that we added the additional "--configuration=jumpy" to  *cmdline* of the jumpy setting.

#### Deleting unused lines
In this example, we are trying to run benchmarks on a SLURM machine. As mentioned in the general guide, pbsjob is the appropriate choice. We delete line 13 and lines 21-23 since they refer to seqjob.

#### Adapting pbsjob
Line 15  can be mostly left as it is. The one change we are making is adding an explicit reference to *partition* and setting its value to "long".
```
<pbsjob name="pbs-gen" timeout="1200" runs="1" script_mode="timeout" walltime="23:59:59" cpt="4" partition="long"/>
```

#### Project changes
The last changes needed are to lines 25-27. First, we want to name our project something meaningful. We set the value of *name* to "base-vs-jumpy"

The value of *tag* is no longer referring to anything. Both settings that we have different tags but we still want to run both in this benchmark. To manage this we simply set the value of *tag* to the special value \*all\*
```
<project name="base-vs-jumpy" job="pbs-gen">
	<runtag machine="zuse" benchmark="no-pigeons" tag="*all*"/>
</project>
```

## Running it on the cluster
For this part, we assume that you have cloned the benchmark tool and that the modified runscript was saved into the runscript folder of the benchmark tool.

The first thing we have to do is to copy our "clingo-vanilla" script into the programs folder and make sure that it is executable. Following the instruction of the general guide, we now run the bgen script.
```
$ ./bgen runscripts/runscript-vanilla-clingo.xml 
```

This creates a folder named "vanilla-clingo". Now, we can execute the start script:
```
$ ./vanilla-clingo/base-vs-jumpy/zuse/start.sh
```

This starts the benchmark. We can check that they are running with the "squeue" command.

When the benchmarks have finished running we can now run the beval script.
```
./beval runscripts/runscript-vanilla-clingo.xml > vanilla-eval.xml
```

Finally, we run the bconv script on the evaluation xml file.
```
$ ./bconv -m time:t vanilla-eval.xml > results.ods
```

The table with the time for the instances is now saved in "results.ods"