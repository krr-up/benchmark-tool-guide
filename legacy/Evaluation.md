In this document we take a closer look into the evaluation scripts for the benchmark tool. The evaluation script is very important to us because it is what will retrieve whatever information we want/need from the output of the benchmarks. There are some scripts already available, such as [clasp.py](https://github.com/potassco/benchmark-tool/blob/master/src/benchmarktool/resultparser/clasp.py) that can grab many of clingo's statistics. However, sometime we will need access to other statistics that the script doesn't retrieve. We could also be running a programm that has its own set of statistics.

We will now take a look at how the scripts work, how to change them to fit your own goals and how to write your own.

### Evaluation scripts

As the example script, we will look at [clasp.py](https://github.com/potassco/benchmark-tool/blob/master/src/benchmarktool/resultparser/clasp.py).

First, we take a look at the naming scheme. Inside the file, there is a single function definition. This function **must** have the same name as the file. The function also takes 3 arguments. The first argument is the path to the root directory of the benchmark and where the results are saved. The second argument, *runspec*, gives us access to the data that we defined in the *runscript* file. The third argument is an instance class that includes information such as the location and the name of the instance.

The function is applied to every "run" individually. It gathers the data for a particular run in basically three steps. First, it reads the relevant output files. Secondly, it parses the text in those files to find the data we need. Third, it saves the data in the correct format.

Reading the files takes place in lines 32 and 33. Line 32 loops through the relevant files and line 33 reads the file line by line.
```
    for f in ["runsolver.solver", "runsolver.watcher"]:
        for line in codecs.open(os.path.join(root, f), errors='ignore', encoding='utf-8'):
```
Since we are using a runsolver to run the benchmarks, the results of the run are saved into a file named "runsolver.solver". The statistics that the runsolver keeps track of(e.g. memory usage and time) are saved into a "runsolver.watcher" (The name of these files is defined in the run template). 

To parse the files we make use of regular expressions. In line 12, we define a dictionary that contains several regular expressions that are able to parse a specific statistic. The key of the dictionary is the name of the statistic we want. The value is a tuple where the first value is the type of the data that is parsed and the second value is the regular expression that can parse the value. We make use of this dictionary in lines 34 to 36. For each of the files we loop through, we apply each regular expression in the dictionary to the whole text. If a match is found for any regex then we save the result into the variable *res*. The res variable is also a dictionary with a similar structure as the regex dictionary. The difference is that res has the actual parsed value in the tuple instead of the regular expression. 

After looping through the files and applying the regular expressions we proceed to build the actual *result* variable that we will return. The returned variable **MUST** be a list of triples where each triple has the format (name, data type, data).

We are not limited to returning the values that we parse with the regex. We can do some extra work to extrapolate some other data or just simply return the data in a different format. For example, we could return an integer for each of the several "status" values. So, instead of return a "SATISFIABLE" string we could return a 1. In the same manner we could return 0 instead of "UNSATISFIABLE" and 2 instead of "OPTIMUM".


### Adding your own statistics

Adding more statistics to parse is fairly simple. We will ilustrate how with an example. Supposing that our output includes the following line:
"Time to run function:	20.23s"

The first step is to add an entry to the regex dictionary. The entry would look as follows:
```
"function_time" : ("float", re.compile(r"^Time to run function:[ ]*(?P<val>[0-9]+(\.[0-9]+)?)$"))
```
Once we add an entry it will always be applied to the files that we read and the match saved in the res variable. The second step would be to read the entry from the res variable as save it into the result variable in the correct format. This is already implemented in line 58. Every entry into the res variable is converted into the correct format and added to the result list.

Suppose that we don't really care that much about how long the function runs, we only care that the time it takes is less than half of the total clingo runtime. In this case, we have to do some extra processing. After the regexes are applied to the files, we will have a "function_time" entry in the res variable. Using this, we can grab that value and also the value of the clingo runtime via the "time" key. Now, we can compare the values:

```
if (res["time"][1] / res["function_time"][1]) < 2:
	result.append("function_time", "string", "acceptable")
else:
	result.append("function_time", "string", "unacceptable")
```

Afterwards, we have to delete the "function_time" entry from "res" so that it is not added to results later on when line 58 is executed:
```
del res["function_time"]
```