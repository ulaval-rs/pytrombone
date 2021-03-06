# pytrombone
Python wrapper for the Trombone project


https://github.com/voyanttools/trombone

## Installation
```
$ pip install pytrombone
```

## Usage
### Examples
Consider a situation where we have a bunch of pdfs in a directory named './data/',
and we want to calculate the SMOG index on those PDFs.

#### Making sure that Trombone works
```python
from pytrombone import Trombone, Cache, filepaths_loader

# This will download the Trombone jar in the /tmp/ directory of your machine. 
# Note that Trombone is likely to be deleted on reboot, and will need to be downloaded again.
trombone = Trombone()

# To get the version
version = trombone.get_version()
print(version)
```

#### Calculating the SMOG index of 2 files
```python
# To run Trombone on a single file use the run method.
# Note that Trombone parameters are given in the form of a list of tuple of 2 elements.
# The first element of the tuple is the parameter, and the second is its value.
# Also note that Trombone will handle those 2 files concurrently 
# (it will be more performant to give many files at the same time rather than loop on each).
output, error = trombone.run([
    ('tool', 'corpus.DocumentSMOGIndex'),  # Choose the tool you want to use
    ('file', './data/example1.pdf'),
    ('file', './data/example2.pdf'),
    ('storage', 'file'),  # Optional, it allows Trombone to cache pre-processed files (use if you will use the file for many tools)
])
output  # is the successful output of Trombone, in the form of a string
error  # is the failed output of Trombone, in the form of a string

# You can serialize the output, which has the JSON format :
output = trombone.serialize_output(output)
# output is now your results in the form of a dictionary
```

#### Calculating the SMOG index in batches

```python
# We first need to setup the cache file (it will allow you to re-run
# your code in case of a problem without having to restart from the beginning)
cache = Cache('./cache.db')

# Then, load the filepaths in batch. pytrombone has a function to do that.
# Note that every file marked as processed will be ignored.
# Also note that the Cache uses the filename of the file as reference.
for filepaths in filepaths_loader('./data/*.pdf', 100, cache):
    # Making tuples to fit the specification of Trombone parameters
    files = [('file', filepath) for filepath in filepaths]

    output, error = trombone.run([
        ('tool', 'corpus.DocumentSMOGIndex'),  # Choose the tool you want to use
        ('storage', 'file'),  # Optional, it allows Trombone to cache pre-processed files (use if you will use the file for many tools)
    ] + files)

    try:
        # If the serialization failed, it is because Trombone failed to performs the analysis.
        # the failed files will be marked as failed in the cache and re-run on the next run.
        # You may want to inspect the "error" variable for more information.
        output = trombone.serialize_output(output)
    except json.JSONDecoder:
        filenames = [os.path.basename(filepath) for filepath in filepaths]
        cache.mark_as_failed(filenames)
        continue

    output  # now has your results in the for of a dictionary
```
