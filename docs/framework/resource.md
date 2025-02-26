---
script:
  basepath: data
topics:
  selector: h2
---

# Resource Class

The Resource class is arguable the most important class of the whole Frictionless Framework. It's based on [Data Resource Standard](https://specs.frictionlessdata.io/data-resource/) and  [Tabular Data Resource Standard](https://specs.frictionlessdata.io/data-resource/)

## Creating Resource

Let's create a data resource:

```python tabs=Python
from frictionless import Resource

resource = Resource('table.csv') # from a resource path
resource = Resource('resource.json') # from a descriptor path
resource = Resource({'path': 'table.csv'}) # from a descriptor
resource = Resource(path='table.csv') # from arguments
```

As you can see it's possible to create a resource providing different kinds of sources which will be detector to have some type automatically (e.g. whether it's a descriptor or a path). It's possible to make this step more explicit:

```python tabs=Python
from frictionless import Resource

resource = Resource(path='data/table.csv') # from a path
resource = Resource('data/resource.json') # from a descriptor
```

## Describing Resource

The standards support a great deal of resource metadata which is possible to have with Frictionless Framework too:

```python script tabs=Python
from frictionless import Resource

resource = Resource(
    name='resource',
    title='My Resource',
    description='My Resource for the Guide',
    path='table.csv',
    # it's possible to provide all the official properties like mediatype, etc
)
print(resource)
```

If you have created a resource, for example, from a descriptor you can access this properties:

```python script tabs=Python
from frictionless import Resource

resource = Resource('resource.json')
print(resource.name)
# and others
```

And edit them:

```python script tabs=Python
from frictionless import Resource

resource = Resource('resource.json')
resource.name = 'new-name'
resource.title = 'New Title'
resource.description = 'New Description'
# and others
print(resource)
```

## Saving Descriptor

As any of the Metadata classes the Resource class can be saved as JSON or YAML:

```python tabs=Python
from frictionless import Resource
resource = Resource('table.csv')
resource.to_json('resource.json') # Save as JSON
resource.to_yaml('resource.yaml') # Save as YAML
```

## Resource Lifecycle

You might have noticed that we had to duplicate the `with Resource(...)` statement in some examples. The reason is that Resource is a streaming interface. Once it's read you need to open it again. Let's show it in an example:

```python script tabs=Python
from pprint import pprint
from frictionless import Resource

resource = Resource('capital-3.csv')
resource.open()
pprint(resource.read_rows())
pprint(resource.read_rows())
# We need to re-open: there is no data left
resource.open()
pprint(resource.read_rows())
# We need to close manually: not context manager is used
resource.close()
```

At the same you can read data for a resource without opening and closing it explicitly. In this case Frictionless Framework will open and close the resource for you so it will be basically a one-time operation:

```python script tabs=Python
from frictionless import Resource

resource = Resource('capital-3.csv')
pprint(resource.read_rows())
```

## Reading Data

The Resource class is also a metadata class which provides various read and stream functions. The `extract` functions always read rows into memory; Resource can do the same but it also gives a choice regarding output data. It can be `rows`, `data`, `text`, or `bytes`. Let's try reading all of them:

```python script tabs=Python
from frictionless import Resource

resource = Resource('country-3.csv')
pprint(resource.read_bytes())
pprint(resource.read_text())
pprint(resource.read_cells())
pprint(resource.read_rows())
```

It's really handy to read all your data into memory but it's not always possible if a file is really big. For such cases, Frictionless provides streaming functions:

```python script tabs=Python
from frictionless import Resource

with Resource('country-3.csv') as resource:
    pprint(resource.byte_stream)
    pprint(resource.text_stream)
    pprint(resource.cell_stream)
    pprint(resource.row_stream)
    for row in resource.row_stream:
      print(row)
```

## Scheme

The scheme also know as protocol indicates which loader Frictionless should use to read or write data. It can be `file` (default), `text`, `http`, `https`, `s3`, and others.

```python script tabs=Python
from frictionless import Resource

with Resource(b'header1,header2\nvalue1,value2', format='csv') as resource:
  print(resource.scheme)
  print(resource.to_view())
```

## Format

The format or as it's also called extension helps Frictionless to choose a proper parser to handle the file. Popular formats are `csv`, `xlsx`, `json` and others

```python script tabs=Python
from frictionless import Resource

with Resource(b'header1,header2\nvalue1,value2.csv', format='csv') as resource:
  print(resource.format)
  print(resource.to_view())
```

## Encoding

Frictionless automatically detects encoding of files but sometimes it can be inaccurate. It's possible to provide an encoding manually:

```python script tabs=Python
from frictionless import Resource

with Resource('country-3.csv', encoding='utf-8') as resource:
  print(resource.encoding)
  print(resource.path)
```
```
utf-8
data/country-3.csv
```

## Innerpath

By default, Frictionless uses the first file found in a zip archive. It's possible to adjust this behaviour:

```python script tabs=Python
from frictionless import Resource

with Resource('table-multiple-files.zip', innerpath='table-reverse.csv') as resource:
  print(resource.compression)
  print(resource.innerpath)
  print(resource.to_view())
```

## Compression

It's possible to adjust compression detection by providing the algorithm explicitly. For the example below it's not required as it would be detected anyway:

```python script tabs=Python
from frictionless import Resource

with Resource('table.csv.zip', compression='zip') as resource:
  print(resource.compression)
  print(resource.to_view())
```

## Dialect

The Dialect adjusts the way the parsers work. The concept is similar to the Control above. Let's use the CSV Dialect to adjust the delimiter configuration:

```python title="Python"
from frictionless import Resource
from frictionless.plugins.csv import CsvDialect

source = b'header1;header2\nvalue1;value2'
dialect = CsvDialect(delimiter=';')
with Resource(source, format='csv', dialect=dialect) as resource:
  print(resource.dialect)
  print(resource.to_view())
```
```
{'delimiter': ';'}
+----------+----------+
| header1  | header2  |
+==========+==========+
| 'value1' | 'value2' |
+----------+----------+
```

There are a great deal of options available for different dialects that can be found in "Formats Reference". We will list the properties that can be used with every dialect:

## Schema

Please read [Schema Guide](schema.html) for more information.

## Stats

Resource's stats can be accessed with `resource.stats`:

```python script tabs=Python
from frictionless import Resource

resource = Resource('table.csv')
resource.infer(stats=True)
print(resource.stats)
```

## Reference

```yaml reference
references:
  - frictionless.Resource
  - frictionless.Loader
  - frictionless.Parser
```
