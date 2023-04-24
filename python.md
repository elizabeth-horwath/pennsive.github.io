# Intro to Python

In this tutorial, we'll be covering some of the basics of how to use Python to do whatever it is you need to do, including:  

*   Python vs. R
*   Packages and evironments
*   Compilers
*   Using Python in R
*   Using R in Python
*   Dataframe manipulation 
*   Visualization 






# Python vs. R

Both Python and R are powerful languages for scientific computing, but with some key differences. The biggest is that while both are publicly available, the syntax is different and there are some packages that can be found in one that can't be found in the other. The latter is why I often find myself switching between the two. Some other differences are:

**R** was developed for statistical programming, and its capabilities reflect that. There are many statistical packages for R that don't have Python equivalents, and R has many more (arguably nicer) graphics for statistical analyses (e.g. ggplot). 

**Python** is a more all-purpose programming language that can be used for data science and other programs. While it can also be used for data analyses, many statistical methods and data manipulation tools have to be loaded in as packages (as opposed to being built-in in R). I've also found that many machine and deep learning tools were originally developed in Python, although R equivalents are fast becoming available. 

*Which one should I use?*

This depends on both you and the project! You can pick the language that has the packages you need to run analyses for your project, or whichever language better fits your personal style and preferences. 

# Packages and Environments

## What is a package?

In the same way an R package contains all the functions to complete tasks, Python packages function the same way. There's two ways of handling this: *modules* and *packages*.

A **module** is a .py file containing the functions and classes you want to use that you can import into your code. For example, if you had "hello.py" containing the function:
```python
def hello(n): 
  for i in range(n): 
    print('hello world') 
```
You can import this function into another .py file:
```python
import hello

hello.hello(10)
```

A **package** is a group of multiple modules packaged together. Most of the publicly available ones are hosted on PyPI (the Python Package Index), but there are some still in development that are hosted on GitHub. You can install packages from either, as well as develop your own!



## Installing packages

### PyPI

Most (if not all) commonly used packages are hosted on PyPI. You can directly install packages using the "pip" command. As a general overview of the syntax, you install package (on Windows) using:
```sh
pip install packagename
```

For more information on how to install specific versions of a package, uninstalling packages, and more, you can look at the [PyPI documentation](https://pip.pypa.io/en/stable/user_guide/)

### GitHub

If the package can't be found on PyPI and is instead hosted on GitHub, you can still install it using pip syntax:

```sh
pip install --upgrade https://github.com/user/package
```

## Environments (Conda, Pipenvs, etc.)

The idea of an **environment** is important for reproducible programming. An environment can be thought of as the software and packages (with specific version numbers) that you use to code a particular task. For example, using <code>pandas 1.0.2</code> and <code>numpy 1.4.2</code>  in <code>Python 2.8</code>to write <code>hello.py</code>. This is important for reproducible programming because functions in different package can use different syntax or give slightly different results. Anyone else interested in reproducing your results needs to use the same packages as you to achieve the same results. 


Most environment managers accomplish much the same thing, but some compilers will prefer a particular method (for example, Anaconda compilers function better with conda).

### Conda

**Conda** is a generalized environment and package manager that works over many different languages, including Python. Conda can be used to install packages (similar to pip) AND to generate different environments for different codes you want to run or develop. For example, if you wanted to use <code>numpy 1.4.2</code> for <code>code1.py</code> and <code>numpy 1.2.1</code> for <code>code2.py</code>, you can create two environments (one for each version) and run each code in its corresponding environment. 

Environments can also be shared with others for reproducibility by exporting the <code>.yaml</code> file associated with each environment.

For more documentation on the specifics of conda syntax, you can read the [conda documentation](https://docs.conda.io/projects/conda/en/latest/commands.html).

### Pipenv and Venv

Pip and virtual environments are also efficient methods for managing environments and packages. 

**Virtualenv** or **venv** (virtual environment) is the lightweight version of pipenv, where you can activate an environment and install packages such that version will be saved in a <code>requirements.txt</code> file for others to use. The only difference between the two is that venv comes with Python3, while virtualenv has to be installed separately. 

**Pipenv** is built on top of pip that allows you to pip install specific packages into an environment, then updates a pipfile with all the packages you're using. It essentially combines aspects of pip and virtual environments into a single system.

# Compilers

While there are many possible Integrated Development Environments (IDEs) for Python code, the three I tend to use the most are Spyder, Jupyter Notebook, and PyCharm. All three provide interactive environments for coding in Python similar to how RStudio acts as the user interface for R. The choice of IDE is completely up to personal preference, and may depend on your project needs.

A coding concept in Python that you should be aware of as you use a compiler is a **kernel**, which is similar to a workspace in R. Just as in a workspace, a kernel will contain all the variables and packages that you've loaded. To clear the workspace, you need to restart the kernel. All three of the compilers discussed below allow you to have multiple kernels simultaneously, so be aware of what kernels you have running and what's in each kernel. 

## Spyder

Spyder is the IDE that comes with installation of the Anaconda Navigator. For context, the Anaconda Navigator is a GUI set up as the interface for conda environments, Python, and the user. Because of this, Spyder can be run out of **conda environments** set up through the Anaconda Navigator or the command line.

Coding in Spyder is most similar to coding R scripts in RStudio, where you can run code in the console and view variables as well as generated graphics. Jupyter notebooks are also integrated with Spyder to enable easy interfacing between the two.

## PyCharm

PyCharm is an IDE developed by JetBrains that is similar to Spyder but with a different interface and with more advanced tools for code analysis. The most relevant big difference between PyCharm and Spyder is that the default is to use **pipenvs and venvs** to set up environments. 

Jupyter notebooks can also be run out of and integrated into PyCharm.

## Jupyter Notebook

Jupyter Notebook is the most interactive of the IDEs, and is most similar to writing code in RMarkdown. Just as in RMarkdown, you can mix chunks of code with markdown text and visualize results in real-time. 

The one tradeoff is that there is no interface for viewing variables, so debugging becomes more difficult. If you need to more thoroughly analyze the contents of your variables for any reason, it is recommended that you use PyCharm or Spyder. 

# Using Python in R

It can often be more seamless to have both R and Python code in the same script, and luckily packages exist in R that allow usage of Python syntax in RStudio.

## reticulate

The package ["reticulate"](https://rstudio.github.io/reticulate/) is the most popular option for interfacing between R and Python. This interfacing includes importing and calling Python functions as well as passing data between R and Python, and comes with support for Python environments (see next section). There's two ways of using reticulate:

**1. R Scripting**
To call Python functions in an R script, you can use the following syntax:
```R
library(reticulate)

# Import the "math" Python package
py = import("math") 

# Print "math" implementation of pi
py$pi
```

You can use the package to essentially run Python in the middle of your R code.

**2. R Markdown**

This package is already integrated into R Markdown such that if you click the "Add Code Chunk" button in RStudio it'll automatically embed a Python code chunk in the middle of your RMD code that should look something like this (ignoring the comment signs of course):
```sh
#```{python}
#print(1)
#```
```



# Using R in Python

Just as you can use Python in R, you can also do the opposite!

## rpy2

The "rpy2" package for Python can be used to interface with R packages, enabling usage of CRAN and other R packages not found in Python. This can be used in both Python scripts and in Jupyter notebooks, and an example is shown below:

```python
# For importin R datatypes
import rpy2.robjects as robjects
# For importing packages
from rpy2.robjects.packages import importr 

base = importr("base")
stats = importr("stats")

xs = robjects.FloatVector([1, 2, 3, 4, 5])
```

For more information, you can check out the [rpy2 official documentation](https://rpy2.github.io/doc/v3.5.x/html/index.html).

# Dataframe Manipulation

The main workhorse for analyzing data is the ability to manipulate data tables, or "dataframes". As a statistical programming language, R has many built-in functions for dataframe manipulation (e.g. data.frame is built-in). Python requires packages for manipulating data tables, of which the most popular are **numpy** and **pandas**. 

More information on the exact syntax can be found in the provided links to the documentation.

## numpy

The **numpy** package is essentially the package for **numeric matrix operations** in Python. You'll find that numpy is included as a dependency in many packages for this reason, and unning matrix operations will often require converting dataframes into numpy matrices. In addition, numpy matrices must all be of the same data type, unlike pandas dataframes, and indexing is only integer-based.

For more information on numpy syntax, see the [documentation](https://numpy.org/doc/).

## pandas

The **pandas** package is essentially the package for **data manipulation**. This package provides utilities such as reading/writing data in spreadsheets, merging/grouping dataframes, and more. The pandas DataFrame object is more or less equivalent to the data.frame object in R. Just as in a data.frame object, the pandas DataFrame can contain multiple data types and can be indexed by labels. 

It is important to note that a pandas **Series** is distinct from a pandas **DataFrame**. A Series object will only have one column, while a DataFrame object can have one or more columns. Certain functions will only work for objects of either a Series or DataFrame type, so be aware of which one you're working with.

For more information on pandas syntax, see the [documentation](https://pandas.pydata.org/docs/).

# Visualization

Python also has many libraries for visualizing data, but the two most commonly used packages are **seaborn** and **matplotlib**. While I won't go into too much detail on the exact syntax (instead I'll provide references to the documentation, which should have more than enough detail), I'll give a high-level explanation of the differences between the two.

In general, while seaborn and matplotlib provide a lot of coverage, there are many plots in R that you can't create in Python (for example, there's no faceting in Python). This is why R is still the language of choice for statisticians to visualize data. However, matplotlib and seaborn still are a great option for visualizing the results of your Python code. There are also some task-specific visualization packages (e.g. Tensorboard for Tensorflow neural nets) that exist in Python that are not in R.

## seaborn

The package **seaborn** is a higher-level API that is faster, simpler to use, and has a better default aesthetic. However, because it is simpler there is a more limited range of available plot types and it's harder to manipulate specific components of the plot. 

For more information on seaborn syntax, see the [documentation](https://seaborn.pydata.org/tutorial.html).

## matplotlib

The package **matplotlib** is a lower-level API that has more available plot types and enables customization of just about every element on the plot. However, because there are more options usage is not as easy as in seaborn, and getting your plot just right can take longer. 

For more information on matplotlib syntax, see the [documentation](hhttps://matplotlib.org/stable/index.html).

Also, fun fact: matplotlib is based on the visualization syntax for MATLAB, so if you've used MATLAB before the syntax is very similar!
