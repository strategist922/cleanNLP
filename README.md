[![CRAN Version](http://www.r-pkg.org/badges/version/cleanNLP)](https://CRAN.R-project.org/package=cleanNLP)  [![AppVeyor Build Status](https://ci.appveyor.com/api/projects/status/github/statsmaths/cleanNLP?branch=master&svg=true)](https://ci.appveyor.com/project/statsmaths/cleanNLP) [![Travis-CI Build Status](https://travis-ci.org/statsmaths/cleanNLP.svg?branch=master)](https://travis-ci.org/statsmaths/cleanNLP) [![Coverage Status](https://img.shields.io/codecov/c/github/statsmaths/cleanNLP/master.svg)](https://codecov.io/github/statsmaths/cleanNLP?branch=master) ![Downloads](http://cranlogs.r-pkg.org/badges/cleanNLP)

## A Tidy Data Model for Natural Language Processing

The **cleanNLP** package is designed to make it as painless as possible
to turn raw text into feature-rich data frames. You can download the
package from within R, either directly from CRAN:
```{r}
install.packages("cleanNLP")
```
Or you may grab the development version using devtools:
```{r}
devtools::install_github("statsmaths/cleanNLP")
```
As described in detail below, the package has a bare-bones parser that
does not require any additional system dependencies. However, to benefit
from the package most users will want to load either a Python or Java
backend. Detailed instructions for doing this are included below.

## Basic usage

We take as an example the opening lines of Douglas Adam's
*Life, the Universe and Everything*:

```{r}
text <- c("The regular early morning yell of horror was the sound of",
          "Arthur Dent waking up and suddenly remembering where he",
          "was. It wasn't just that the cave was cold, it wasn't just",
          "that it was damp and smelly. It was the fact that the cave",
          "was in the middle of Islington and there wasn't a bus due",
          "for two million years.")
text <- paste(text, collapse = " ")
```

A minimal working example of using **cleanNLP** consists of loading the
package, setting up the NLP backend, initalizing the backend, and running
the function `run_annotators`. Because our input is a text string we set
`as_strings` to `TRUE` (the default is to assume that we are giving the
function paths to where the input data sits on the local machine"):

```{r}
library(cleanNLP)
init_spaCy()
obj <- run_annotators(text, as_strings = TRUE)
```

Here, we used the spaCy backend. A discussion of the various backends that
are available are given the following section. The returned annotation
object is nothing more than a list of data frames (and one matrix),
similar to a set of tables within a database. The names of these tables
are:

```{r}
names(obj)
```
```
## [1] "coreference" "dependency"  "document"    "entity"      "sentence"
## [6] "token"       "vector"
```

The canonical way of accessing these data frames is by using functions of
the form `get_TABLE`. For example, the document table gives metadata about
each document in the **corpus**, which here consists of only a single
document:

```{r}
get_document(obj)
```
```
# A tibble: 1 × 5
     id                time version language
  <int>              <dttm>   <chr>    <chr>
1     1 2017-05-20 15:24:44   1.8.2     <NA>
# ... with 1 more variables: uri <chr>
```

The tokens table has one row for each word in the input text, giving data
about each word such as its lemmatized form and its part of speech. We
access these table with the `get_token` function:

```{r}
get_token(obj)
```
```
# A tibble: 68 × 8
      id   sid   tid    word   lemma  upos   pos   cid
   <int> <int> <int>   <chr>   <chr> <chr> <chr> <int>
1      1     1     1     The     the   DET    DT     0
2      1     1     2 regular regular   ADJ    JJ     4
3      1     1     3   early   early   ADJ    JJ    12
4      1     1     4 morning morning  NOUN    NN    18
5      1     1     5    yell    yell  NOUN    NN    26
6      1     1     6      of      of   ADP    IN    31
7      1     1     7  horror  horror  NOUN    NN    34
8      1     1     8     was      be  VERB   VBD    41
9      1     1     9     the     the   DET    DT    45
10     1     1    10   sound   sound  NOUN    NN    49
# ... with 58 more rows
```

The output from the `get` functions are (mostly) pre-calculated. All of the hard
work is done in the `run_annotators` function.

## Backends

There are three "backends" for parsing text in **cleanNLP**. These are:

- an R-based version using only the **tokenizers** package. It offers minimal
output but requires no external dependencies.
- a Python-based parser using the [spaCy](https://spacy.io/) library.
Requires installing Python and the library separately; this is generally
pain free and works with any version of Python >= 2.7. The library is very
fast and provides the basic annotators for parsing text such as lemmatization,
part of speech tagging, dependency parsing, and named entity recognition.
There is also support for computing word embeddings in English.
- a Java-based parser using the [CoreNLP](http://stanfordnlp.github.io/CoreNLP/)
library. Setting this up is often more involved and the data files are quite
large. The pipeline also takes significantly longer than the spaCy implementation.
However, the CoreNLP parser offers many more bleeding-edge annotation tasks
such as coreference resolution and speaker detection.

The only benefit of the R-based version is its lack of external dependencies.
We supply it mainly for testing and teaching purposes when access to the machine(s)
running the code are not under our control. For most use-cases we recommend using
the spaCy library as it strikes a balance between features, speed, and ease of set-up.
The CoreNLP backend should be used when access to the advanced annotation tasks
is required or when parsing in languages not yet available in spaCy.

### spaCy backend (python)

To use the Python based backend spaCy, users must first install an up to
date version of Python. For users without prior experience, we recommend
installing the Python 3.6 version of
[Anaconda Python](https://www.continuum.io/downloads). Anaconda has both
command line and GUI installers for all major platforms; these include
many of the basic scientific Python modules (these can be troublesome
to install otherwise). Make sure that you have at least version 3.6.1
or greater. Of particular note, we do not recommend or support
using the default system installation of Python that is bundled with
MacOS, Ubuntu, and many other linux distributions. This version is often
out-of-date, almost impossible to update, and causes permissions issues
when installing modules.

Once Python is installed, the next step is to install the spaCy module
and its associated models. There are detailed instructions for doing
this on the [spaCy website](https://spacy.io/docs/usage/) that support
many different platforms and versions of Python. Note that on windows
you must also have a working version of Virtual Studio Express
(preferably the 2015 version). If you installed the
most recent version of Anaconda Python 3.6 as directed above, simply
run the following commands in the terminal (Mac/Linux) or shell (Windows):

```
conda config --add channels conda-forge
conda install spacy
python -m spacy download en
```

The last line downloads the English models ("en"); repeat or replace
with French ("fr") or German ("de") based on your needs.

Finally, the R package **reticulate** must also be installed, which
can be downloaded from CRAN:
```{r}
install.packages("reticulate")
```

And, before running any cleanNLP command, run `use_python` to set the
location of the Python executable. If using Anaconda Python on a Mac
with the default settings, for example, the following should be correct:

```{r}
use_python("/Users/USER/anaconda3/bin/python")
```

Generally, we have found that the only difficult step in this process
is installing spaCy. Getting Python is almost always issue free as well
as install the **reticulate** package. On a Mac, we've found that best
solution is to start with a fresh version of Anaconda 3.6 (the 2.7
series causes problem even though it technically should work). We have
not had an issue getting the library installed when doing this.
On Windows, a fresh version of Anacond also helps, but even then there
can be futher issues. On Windows, it seems that there is no
general solution that works for everyone, unfortunately. Your best bet
is to follow the instructions above and then look up your particular
warning on the [spaCY issues page](https://github.com/explosion/spaCy/issues).

After the backend is install, you should be able to run the code in the
preceeding section "Basic usage" as given.

### coreNLP backend (Java)

In order to make use of the Java-based coreNLP backend, a version of
Java >= 7.0 must be installed and the **rJava** package must be set up.
This should be straightforward, and generally runs without issue on
Linux and Windows. On Mac, there are issues arising from conflicts with
the system version of Java. The detailed instructions
[Problem With rJava On Mac OS X El Capitan](http://charlotte-ngs.github.io/2016/01/MacOsXrJavaProblem.html) from
from Peter von Rohr have solved these issues on all of our test systems.
For additional help, see the GitHub issues tracker and submit any new
bugs that arise.

Once these system requirements are met, we can install the ".jar" files
inside of R with the following function:

```{r}
download_core_nlp()
```

These files are large and may take several minutes to download. The Java
backend is then configured with `init_coreNLP`. The easiest
interface is to specify an annotation level from 0 to 3, with higher numbers
including more models but taking increasingly long to parse the text.
Setting it equal to 2 is a good balance between time and
feature-richness:

```{r}
init_coreNLP(anno_level = 2L, lib_location = lib_loc)
```

After the pipeline is loaded, we again call run_annotators and set the
backend to "coreNLP" (by default run_annotators will use whichever backend
for most recently initalized, so this option is technically not
needed if you just ran `init_coreNLP`):

```{r}
obj <- run_annotators(text, as_strings = TRUE, backend = "coreNLP")
obj
```
```
##
## A CleanNLP Annotation:
##   num. documents: 1
```

The annotation object contains the same tables as the spaCy models,
with slightly different fields filled in.

## Saving annotations

Once an annotation object is created there are two ways of saving the output.
Using `saveRDS` saves the output as a binary file which can be read back into
R at any time. Alternatively, the function `write_annotation` saves the annotation
as a collection of comma separated files:

```{r}
od <- tempfile()
write_annotation(obj, od)
dir(od)
```
```
[1] "dependency.csv" "document.csv"   "entity.csv"     "token.csv"
[5] "vector.csv"
```

Notice that only those tables that are non-empty are saved. These may be
read back into R as an annotation object using `read_annotation`:

```{r}
anno <- read_annotation(od)
```

Alternatively, as these are just comma separated values, the data may be read
directly using `read.csv` in R or whatever other programming language or software
a user would like to work with.

## More information

This document is meant just to get users up to speed enough to starting being
useful with the **coreNLP** package. There are many more options and helper
functions available to make working with textual data as easy as working with
tabular datasets. For a full example of using the package to do an analysis
of a corpus see the vignette:

 - [Exploring the State of the Union Addresses: A Case Study with cleanNLP](https://cran.r-project.org/package=cleanNLP/vignettes/case_study.html).

For more detailed information about the fields in each of the tables, users
can consult the help pages for the `get_` functions. An even more detailed
guide to the fields and the underlying philosophy is given in the vignette

- [A Data Model for the NLP Pipeline](https://cran.r-project.org/package=cleanNLP/vignettes/schema.html).

## Note

Please note that this project is released with a [Contributor Code of Conduct](CONDUCT.md). By participating in this project you agree to abide by its terms.


