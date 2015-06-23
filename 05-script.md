# Command line programs in Python (aka Python scripts)

## Creating a Python script

The IPython Notebook and other interactive tools are great for prototyping code and exploring data, but sooner or later we will want to use our program in a pipeline or run it in a shell script to process thousands of data files. In order to do that, we need to make our programs work like other Unix command-line tools. For example, we may want a program that reads a data set, performs the fit and saves both the plot and the output fit parameters to files.

First of all, let's create another `.py` file named `analyze_mosquito_data_script.py` in our repository folder
and let's copy the code we used in the last class on Python to run the functions defined in `analyze_mosquito_data_lib.py` on our data.

```python
    import pandas as pd
    import analyze_mosquito_data_lib as mosquito_lib

    filename = "A1_mosquito_data.csv"
    data = pd.read_csv(filename)
    parameters = mosquito_lib.analyze(data)
    print(parameters)
```

Now we can run this script in batch mode from the command line, without opening the IPython notebook:

    python analyze_mosquito_data_script.py

if this gives you errors due to libraries, there may be something wrong with your path, try:

    ~/anaconda/bin/python analyze_mosquito_data_script.py

or (on Windows):

    ~/Anaconda/Scripts/ipython.exe analyze_mosquito_data_script.py

Python runs this script line by line, somehow similarly to running every cell of an IPython Notebook with the "Run All" button.

What is the difference between a module and a script?

* `analyze_mosquito_data_lib.py` is a module, it is a library that just defines functions, it doesn't actually run any analysis, it is a library that collects functions we can re-use in any number of IPython Notebooks or scripts.
* `analyze_mosquito_data_script.py` is a script, it imports the functions defined in different libraries and executes them to actually run an analysis and produce results.

We want to try keep a script as simple as possible, all of our algorithms should be implemented in a library,
so that they can be re-used, shared and improved over time. Generally the library is what is published online on Github, generally together with some sample scripts to show how to use the library.

## Saving output files

The script only prints the output parameters to the console,
first thing we need to implement saving the parameters to file and saving the plot to disk.

We will modify the module `analyze_mosquito_data_lib.py`.

The `parameters` variable is a data structure defined in `pandas` and has a method to write its own content
to disk as comma separated values, `.to_csv`:

```python
parameters.to_csv("parameters.csv")
```

Saving a plot is accomplished calling:

```python
plt.savefig("output_plot.png")
```
```python
def analyze(data, filename):
"""Perform regression analysis on mosquito data

Performs a linear regression based on rainfall and temperature.
Creates two plots of the results and returns fit parameters.

Parameters
----------
data : pandas.Dataframe
Column named 'temperature', 'rainfall' and 'mosquitos'.

Returns
-------
parameters : list of pandas.Series
Return a list of the fitting parameters for rainfall and temperature

"""
data['temperature'] = fahr_to_celsius(data['temperature'])
assert data['temperature'].min() > -50
assert data['rainfall'].min() > 0
output = []
for variable in ['rainfall', 'temperature']:
# linear fit
regr_results = sm.OLS.from_formula('mosquitos ~ ' + variable, data).fit()
parameters = regr_results.params
parameters.to_csv("parameters.csv")
line_fit = parameters['Intercept'] + parameters[variable] * data[variable]
# plotting
plt.figure()
plt.plot(data[variable], data['mosquitos'], '.', label="data")
plt.plot(data[variable], line_fit, 'red', label="fit")
plt.xlabel(variable)
plt.ylabel('mosquitos')
plt.legend(loc='best')
plt.savefig("output_plot.png")
output.append(parameters)
return output
```

This is not yet ideal. We can not specify the filenames and we are overwritting the rainfall files and plots with the temperature ones.

## Dynamic output filenames

If we modify the `filename` variable in the script, we can run the analysis on other files, however the output plot and `.csv` would be overwritten because they have the same name.

We need to dynamically create the name of the output files.
First of all the `analyze` function needs to get the output figure filename as an input parameter, because at function definition we have no knowledge of what will be the name of the output file.

The final version of the function will be:

```python
def analyze(data, figure_filename):
```

If the input `filename` is `A1_mosquito_data.csv`, we could like the outputs to be `A1_mosquito_data.png` `A1_mosquito_parameters.csv`.
We can achieve this with the `replace` method of the `filename` string:

```python
def analyze(data, filename):
"""Perform regression analysis on mosquito data

Performs a linear regression based on rainfall and temperature.
Creates two plots of the results and returns fit parameters.

Parameters
----------
data : pandas.Dataframe
Column named 'temperature', 'rainfall' and 'mosquitos'.

Returns
-------
parameters : list of pandas.Series
Return a list of the fitting parameters for rainfall and temperature

"""
data['temperature'] = fahr_to_celsius(data['temperature'])
assert data['temperature'].min() > -50
assert data['rainfall'].min() > 0
output = []
for variable in ['rainfall', 'temperature']:
# linear fit
regr_results = sm.OLS.from_formula('mosquitos ~ ' + variable, data).fit()
parameters = regr_results.params
csv_filename = filename.replace("data", "parameters_" + variable)
parameters.to_csv(csv_filename)
line_fit = parameters['Intercept'] + parameters[variable] * data[variable]
# plotting
plt.figure()
plt.plot(data[variable], data['mosquitos'], '.', label="data")
plt.plot(data[variable], line_fit, 'red', label="fit")
plt.xlabel(variable)
plt.ylabel('mosquitos')
plt.legend(loc='best')
plot_filename = filename.replace("csv", "png").replace("data", "plot_" + variable)
plt.savefig(plot_filename)
output.append(parameters)
return output
```

Therefore the current version of `analyze_mosquito_data_script.py` is:

```python
    import pandas as pd
    import analyze_mosquito_data_lib as mosquito_lib

    filename = "A1_mosquito_data.csv"
    data = pd.read_csv(filename)
    mosquito_lib.analyze(data, filename)
```

## Arguments from the command line

Finally we would like to be able to call this script from the command line and choose the target file as:

    python analyze_mosquito_data_script.py B1_mosquito_data.csv

Python support this via the `sys` standard library module:

    import sys
    filename = sys.argv[1]

`sys.argv` is a list of strings, the zeroth element is the name of the actual script, then all the other elements are the other arguments.

The final version of the script is:

```python
import sys
import pandas as pd
import analyze_mosquito_data_lib as mosquito_lib

filename = sys.argv[1]

print "Analyzing", filename

# read the data
data = pd.read_csv(filename)

print "Running analyze"
mosquito_lib.analyze(data, filename)
```
