# LCIA formatter
The LCIA formatter is a Python 3 package for creating LCIA methods from their
original sources by converting them into a [pandas](https://pandas.pydata.org/)
data frame in the [LCIAmethod format](./format%20specs/LCIAmethod.md).

Flow mappings as defined in the
[Fed.LCA.Commons](https://github.com/USEPA/Federal-LCA-Commons-Elementary-Flow-List)
can be applied and the result can be exported to all formats supported by the
`pandas` package (e.g. Excel, CSV) or the
[openLCA JSON-LD format](https://github.com/GreenDelta/olca-schema).

## Usage

In order to use the project, first download it and install it (preferably in a
[virtual environment](https://docs.python.org/3/library/venv.html)):

```bash
$ git clone https://github.com/msrocka/lcia_formatter.git
$ cd lcia_formatter

# create a virtual environment and activate it
$ python -m venv env
$ .\env\Scripts\activate.bat

# install the `master` branch from the Fed.LCA Flow-List repository
pip install git+https://github.com/USEPA/Federal-LCA-Commons-Elementary-Flow-List.git@master

# install the requirements
$ pip install -r requirements.txt

# install the project
$ pip install -e .

# start the Python interpreter
$ python
```

### Loading method data
A data frame with the data of a method can be loaded with the
`get_method(<method ID>)` function

```python
import lciafmt

traci = lciafmt.get_method(lciafmt.Method.TRACI)
```

This will download and cache the raw method data in a temporary folder
(`~/temp/lciafmt`). Before downloading method data, `lciafmt` will first
check if the method data are available in this cache folder. Alternatively,
a file path or web URL can be passed as arguments to the `get_method` function
to load the data from other locations:

```python
import lciafmt

traci = lciafmt.get_method(lciafmt.Method.TRACI,
                           file="path/to/traci_2.1.xlsx")
traci = lciafmt.get_method(lciafmt.Method.TRACI,
                           url="http://.../path/to/traci_2.1.xlsx")
```

Also, it is possible to clear the cache to ensure that the newest version is
downloaded from the internet:

```python
import lciafmt

lciafmt.clear_cache()
traci = lciafmt.get_method(lciafmt.Method.TRACI)
```

The function `supported_methods` returns a list of meta data objects that each
contain the information of LCIA methods that are currently supported:

```python
import lciafmt

lciafmt.supported_methods()
```


### Apply flow mappings
The flow mappings defined in the
[Fed.LCA.Commons](https://github.com/USEPA/Federal-LCA-Commons-Elementary-Flow-List)
can be directly applied on a data frame with method data:

```python
import lciafmt

traci = lciafmt.get_method(lciafmt.Method.TRACI)
traci_mapped = lciafmt.map_flows(traci)
```

This will apply the mapping to the default Fed.LCA.Commons flow list and produce
a new data frame with mapped flows. A specific source system can be selected via
the `system` parameter:

```python
import lciafmt

traci = lciafmt.get_method(lciafmt.Method.TRACI)
traci_mapped = lciafmt.map_flows(traci, system="TRACI2.1")
```

The available systems can be retrieved via the `supported_mapping_systems()`
function:

```python
import lciafmt

lciafmt.supported_mapping_systems()
```

Additionally, the `map_flows` function accepts the following optional parameters:

* `mapping`: a data frame in the
  [Fed.LCA.Commons flow list mapping format](https://github.com/USEPA/Federal-LCA-Commons-Elementary-Flow-List/blob/master/format%20specs/FlowMapping.md)
  that contains the mapping that should be applied
* `preserve_unmapped`: a Boolean value that indicates whether the unmapped flows
  should be preserved in the resulting data frame.


### Data export
The method data are stored in a standard
[pandas data frame](https://pandas.pydata.org/pandas-docs/stable/reference/api/pandas.DataFrame.html)
so that the standard export functions of pandas can be directly used:

```python
import lciafmt

traci = lciafmt.get_method(lciafmt.Method.TRACI)
traci.to_csv("output/traci_2.1.csv", index=False)
```

Additionally, a method can be stored as JSON-LD package that can be imported
into an openLCA database:

```python
import lciafmt

traci = lciafmt.get_method(lciafmt.Method.TRACI)
lciafmt.to_jsonld(traci, "output/traci_2.1_jsonld.zip")
```

When also elementary flows should be written to the JSON-LD package the
`write_flows` flag can be passed to the export call:

```python
import lciafmt

traci = lciafmt.get_method(lciafmt.Method.TRACI)
lciafmt.to_jsonld(traci, "output/traci_2.1_jsonld.zip", write_flows=True)
```

**Note** that unit groups and flow properties are currently not added to the
JSON-LD package so that the package can only be imported into a database where
at least the standard openLCA unit groups and flow properties are available.

### Logging details
The `lciafmt` module writes messages to the default logger of the `logging`
package. In order to see more details, you can set the log level to a finer
level:

```python
import logging as log
log.basicConfig(level=log.INFO)
```

## License
This project is in the worldwide public domain, released under the
[CC0 1.0 Universal Public Domain Dedication License](https://creativecommons.org/publicdomain/zero/1.0/).

![Public Domain Dedication](https://licensebuttons.net/p/zero/1.0/88x31.png)
