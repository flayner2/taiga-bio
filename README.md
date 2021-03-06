# taiga-bio - Package version of the [TaIGa](https://github.com/flayner2/taiga) program

This is a package version of the original TaIGa program. This was built to be available on Pypi and be easily installable
by any Python package manager, and also to allow the user to import and make custom scripts from TaIGa's functionalities.
For better information, see the repo for the original [TaIGa](https://github.com/flayner2/taiga).

## 1 How to run taiga-bio

From a Python script of your choice (must be `>=Python 3.6`), do:

```python
from taiga.core import taxonomy
from taiga.common import data_handlers

taxon_list = taxonomy.run_taiga(input_file, email)
df = data_handlers.create_df(taxon_list)
data_handlers.create_output(ouput_directory, df, taxon_list)

```

This will run TaIGa's main function, which grabs a list of names from `input_file`, fetches for 
their taxonomic information on [NCBI's Taxonomy](https://www.ncbi.nlm.nih.gov/taxonomy), which 
needs your `email` as a good practice, and returns a list of Taxon objects. It then calls the 
`create_df` function, which receives the list of Taxon objects and returns a DataFrame. Then, it 
calls the `create_output` function, which outputs the results to the specified `output_directory`, 
which need not to be pre-created, using the DataFrame and the Taxon list. It needs the Taxon list
to be able to create a file that lists Taxon objects missing critical information.

## 2 Arguments

```python

run_taiga(infile, email, gb_mode=0, tid=False, correction=False, retries=5, silent=False) -> List[Taxon]

```

### 2.1 Positional (required) Arguments:

**[input file]**: This is the full path to the file you will use as an input for TaIGa. By default, TaIGa expects it to be a 
list of organism names separated by line in a text-like file (`.txt`). You can change this behaviour so TaIGa would expect: a line 
separated text file with a collection of Taxon IDs; a Genbank format file with multiple records, all from the same organism; 
a Genbank format file with only one record; or a Genbank format file with multiple records from multiple organisms. Organism names 
refer to any valid taxonomic level that is available on NCBI's Taxonomy database.

**[user e-mail]**: This is just a valid e-mail of yours. Nothing will be sent to this e-mail, and neither TaIGa itself neither me 
will ever use it for anything other than running TaIGa (in fact, I will never have access to this information. You may check the 
code yourself to confirm this). TaIGa only requires this field because it is standard procedure to pass on this information when 
sending requests to Entrez. This is all TaIGa will use the e-mail for. You may pass on gibberish, if you so want, but I advise 
you not to. TaIGa will run fine anyways, as long as you provide something to this argument field.

### 2.2 Optional Arguments:

**gb_mode [0, 1, 2, 3]**: *Default: 0*. This changes TaIGa's default input type to instead expect a Genbank format file. This 
argument exepects one numeric option from the available ones. Those are:

- *0*: Acts the same as not passing the `--gb-mode` argument at all, not altering TaIGa's default behavior.
- *1*: A Genbank format file containing multiple records from multiple, differently named organisms (eg. *Escherichia coli*, 
*Bos taurus*, *Mus musculus*, all in the same `.gb` or `.gbff` file).
- *2*: A Genbank format file containing a single record (eg. an annotation for a *COX 1* gene for *Homo sapiens*).
- *3*: A Genbank format file containing multiple records for a single organism (eg. many annotations for *Apis mellifera* genes).

**tid**: This changes TaIGa's behaviour to, instead of expecting any sort of name-based input, to expect a text file with a list 
of valid Taxon IDs for a collection of organisms (or taxon levels). This is incompatible with the '-c' option, as TaIGa skips the 
spelling correction when run with Taxon IDs. 

**correction**: This enables TaIGa's name correcting functionality. The usefulness of this is discussed below. This is incompatible with 
'--tid'. See '--tid' above.

**retries**: *Default: 5*. This sets the maximum number of retries TaIGa will do when fetching for taxonomic information for an 
organism. This can be very useful as Entrez will many times return broken responses.

**silent**: This disables TaIGa's standard verbose mode, so TaIGa will automatically generate a log file called **TaIGa_run.log** inside
the current working directory. This log file will contain all information about that particular TaIGa run.

## 3 Output files

To create the output files, you'll need to run the `create_output` function from the `taiga.common.data_handlers` module.
It expects an `output_folder`, a `df` and a `taxon_list` as arguments. To create those and output your results, run:

```python
from taiga.core import taxonomy
from taiga.common import data_handlers

taxon_list = taxonomy.run_taiga(input_file, email)
df = data_handlers.create_df(taxon_list)
data_handlers.create_output(ouput_directory, df, taxon_list)

```

The arguments are:

- **[output directory]**: a string containing the path to the output directory, which doesn't need to be created yet.
- **[df]**: the DataFrame returned by `taiga.core.taxonomy.run_taiga()`.
- **[taxon_list]**: the list of Taxon objects returned by `taiga.core.taxonomy.run_taiga()`.

### 3.1 TaIGa_result.csv

After running successfuly, TaIGa will create the output files at the provided output path. If the output folder doesn't exist 
(or its parent folders), TaIGa will check it and create them for you. Do note that you still need to provide a valid path for 
TaIGa to run successfuly. Check it twice before running TaIGa. The created file will be named **TaIGa_result.csv**. This is 
default and can only be changed on the source code, which you can surely do if you know what you're doing. It will be a .csv 
format file. To better visualize the results, import it to any spreadsheet viewer of yours.

The file will contain a number of rows equal to the number of input organisms. Each row will be named for the corresponding taxon.
Each column will be a variable for a particular taxonomic information. The first two are always the organism's Taxon ID and Genome
ID (if it has one). The rest are the valid taxon rank names available on Taxonomy and their corresponding value for each organism.
If there's any missing value (eg., lack of a Genome ID or lack of a *tribe* for *Homo sapiens*), the value will be **N/A**.

### 3.2 TaIGa_missing.txt

TaIGa will also create a file named 'TaIGa_missing.txt'. This will be created regardless if TaIGa was able to run without issues or
any missing information. If any organism happens to be missing one of the core informations TaIGa needs to be able to run (those
being a valid `Name` or `Corrected Name`, `Taxon ID` and `Classification`), that organism will be outputed to this file within the 
correct class of missing information.

## 4 Other functions

TaIGa's `run_taiga()` function calls a bunch of other functions from `helpers`, `parsers`, `fetchers`, `retrievers` and
`data_handlers` and uses the **Taxon** object from `data_models`. You could import those modules with something like:

```python
from taiga.common import fetchers, data_models

animal = data_models.Taxon()
fetchers.fetch_taxonomic_info(email, animal, retries)

```

All modules and functions are nicely documented and you can check their docstrings to see what they do exactly and how
they do it. I won't extend further on them simply because, as of now, they're not really meant to be executed alone.
They will probably work well and do their job if you execute them properly, but individually those are rather simple
wrappers over some common [Biopython](https://biopython.org/) functionalities.

## 5 Licensing 

TaIGa is licensed under the MIT license. You can check the information inside the LICENSE file. To make it short, I wanted it to be
free and open, so that anyone can contribute to it.

## 6 Ending regards

As said in the introduction, the major inspiration for TaIGa's name is the cute romance anime character Taiga, from the japanese 
animation ToraDora. I highly recommend it.

And, as state before, this is simply a package version of the "standalone" original TaIGa python program, that you can
check on [the original repo](https://github.com/flayner2/taiga). There, you will find a better documentation that, albeit
a bit different, is still relevant and might be helpful for this version.

Share love and knowledge and, on top of all, respect people.
