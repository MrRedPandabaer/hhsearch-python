# hhsearch-python
------------------

This small package was made to handle data output by the software suite HHSearch. It was tested with output of the HHSearch version 1.5. The project's idea and draft itself originates from Dr. Schmidt and was done as a final task for one of his university modules.

>HHsearch is a software suite for detecting remote homologues of proteins and generating highquality alignments for homology modeling and function prediction.

[HH-Suite Github](https://github.com/soedinglab/hh-suite) | [Quick Guide to HHSearch](ftp://ftp.tuebingen.mpg.de/pub/protevo/HHsearch/HHsearch1.5.01/HHsearch-guide.pdf)| 

## Installation
-----------------
You can simply install this package through your pip version. 
```sh
pip install hhsearch-python
```
[Find this package on PyPi](https://pypi.org/project/hhsearch-python/)

### Requirements
For full functionalities you need the following packages as well.
```
pandas==0.23.4
matplotlib==3.0.2
numpy==1.15.4
Pillow==6.0.0
pymol==0.1.0
```

Except PyMol, everything can easily be installed through pip install. PyMol needs to be installed separate, as well as being installed through `pip` to be used in your regular Python environment. 

##### ```pip``` installation:
```pip install -c schrodinger pymol``` | ```pip install -c schrodinger pymol```

| PyMol Version | Documentation |
| ------ | ------ |
| MAC | https://pymolwiki.org/index.php/MAC_Install | 
| Windows | https://pymolwiki.org/index.php/Windows_Install | 
| Linux | https://pymolwiki.org/index.php/Linux_Install |

## Functionalities
-----------------

There are a small handful of functions within this package which can be used to generate a decent organized (visualized) output. However, for this all to work properly, you need to have all the needed `.hhm` as well as all `.hhs` files somewhere located in your current working directonary. 

```python
# lets first import all our functions from the module.
from hhsearch_python import *

hhs_file = "data/hhs/d1e0ta1.hhs" # path to your .hhs file.

# first, we can use extract_HHSearch_data() to extract the whole HHSearch statistics into a pandas.DataFrame.
hhs_hits_statistics = extract_HHSearch_data(hhs_file)
```
----
However, we also want regular information about the Query itself, as well as about selected hits. For that we can use the two separate function `extract_HHSearch_main` for the query `.hhs` file, and `get_alignment_term` for a selected hit of the previous created `pandas.DataFrame`.
```python
query_dict = extract_HHSearch_main(hhs_file)

# As an example how this dict() output looks like: 
print(query_dict)
>> {'Query': 'Query d1e0ta1 b.58.1.1 (A:70-167) Pyruvate kinase (PK) {Escherichia coli [TaxId: 562]}',
     'pdb_id': '1e0t',
     'alignment_term': '/1e0t//A/70-167/CA', 
     'full_term': '/1e0t//A//CA', 
     'file_name': 'd1e0ta1'
     }
# alignment_term is needed for a proper PyMol alignment later down the road, 
# as well as full_term, ignoring the specific residues. 

# Let's get information about the second hit of the statistics from the .hhs file. 
hit_dict = get_alignment_term(hhs_hits_statistics, 2)
print(hit_dict)
>> {'pdb_id': '2vgb', 
    'alignment_term': '/2vgb//A/160-261/CA', 
    'full_term': '/2vgb//A//CA', 
    'file_name': 'd2vgba1'}
# except for the key "Query", get_alignment_term() outputs a structure identical dict() as extract_HHSearch_main()
```

Having selected the second alignment as our target-of-choice, we now desire more information about the alignment itself, so we extract the actual alignment with  `get_full_alignment`. It takes two arguments: the `.hhs` file of the query, as well as the number of the hit within the `.hhs` file, just like `get_alignment_term`. So preferably, one looks at the previous created pandas.DataFrame `hhs_hits_statistics` and choses a hit of interest from that. 

```python
# This also creates a html formatted file in a separate folder - /alignments_highlighted/<query>/<NoX-name>.html
# and also the same file as alignment.html in a folder called /lastrun/, all for your convenience. 
alignment_of_interest = get_full_alignment(hhs_file, 2)
```
The HTML formatted output looks like the example below. As you can see, **h**elices and sh**e**ets are colorized. 
>![Example Alignment](https://raw.githubusercontent.com/MrRedPandabaer/hhsearch-python/master/example_alignment.jpg)

Also, if you desire this formatting to be applied on the whole `.hhs` file, then you can use the function `highlight_hhs_full(hhs_file)` and use the path of the desired `.hhs` file as argument. It returns the given hhs file as a colorized html formatted string and also stores within a separate folder `/alignments_highlighted/<query-name>_full.html` as well as in the `/lastrun folder under the filename hhs_full_colorized.html`.

```python
# outputs the whole .hhs file colorized in the above shown pattern. 
full_hhs_colorized = highlight_hhs_full(hhs_file)
```

Having alignments organized and colorized is all useful, but we also want to actually create a more visual representation of the chosen alignment. For that, we can use the previous created dictonaries `query_dict` and `hit_dict` and give their information as arguments to the function `pymol_alignment()`. This function also returns the [rmsd value of atomic positions](https://en.wikipedia.org/wiki/Root-mean-square_deviation_of_atomic_positions) in [ångström](https://en.wikipedia.org/wiki/Angstrom).

```python
# building up the information from the query. 
pdb_1 = query_dict.get("pdb_id")
aln_term_1 = query_dict.get("pdb_id")
full_term_1 = query_dict.get("pdb_id")

# buildung up the needed information from the chosen hit. 
pdb_2 = hit_dict.get("pdb_id")
aln_term_2 = hit_dict.get("pdb_id")
full_term_2 = hit_dict.get("pdb_id")

# Also returns the RMSD values for the alignment. 
rmsd = pymol_alignment(pdb_1, 
                       pdb_2, 
                       aln_term_1, 
                       aln_term_2,
                       full_term_1,
                       full_term_2)

print(rmsd)
>> (0.8026888370513916, 85, 5, 1.2078778743743896, 98, 160.0, 98)
# In this example RMSD Value is about 0.803 Å over 85 C-αlpha atoms. 
```

This will create two images in different folder. One being zoomed-in into the area of `aln_term_1`whih is in our example:  `/1e0t//A/70-167/CA`, showing the area of interest, as well as a non-zoomed in picture of `/1e0t//A//CA` in our example.
These images are stored into the `/lastrun/` folder, as well as in the folder `/PyMol_img/<pdb_1>/<pdb_1>-<pdb_2>/`.
>![main_zoom](https://raw.githubusercontent.com/MrRedPandabaer/hhsearch-python/master/main_zoom.png )

<img src="https://github.com/favicon.ico" width="48">













