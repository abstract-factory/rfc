# Open Metadata Specification

This document defines what it means to be Open Metadata.

![](../images/10/title.png)

* Name: http://rfc.abstractfactory.io/spec/10
* Editor: Marcus Ottosson <marcus@abstractfactory.io>
* Tags: publishing
* State: draft

Copyright and Language can be found in RFC1

# Change Process

This document is governed by the [Consensus-Oriented Specification System](http://www.digistan.org/spec:1/COSS) (COSS).

# History

Open Metadata was first initiated in 2013 to facilitate the development of [Pipi][] and as a response to the ever-more complex nature of metadata for common use.

#### Definition

* meta -- Pertaining to a level above or beyond
* content -- A collection of data
* data -- A piece of information

In layman's terms; "data about data" - regardless of data-type or traditional use.

#### See also

* RFC12: Cascading
* RFC13: Task Distribution
* RFC14: Temporal Metadata
* RFC15: Meta Metadata
* RFC16: Blob
* RFC17: Cross-referencing
* RFC18: Types
* RFC19: Storage Agnostic
* RFC20: Referencing
* RFC35: Garbage Collection
* RFC41: Driver
* RFC46: Temporal Resolution

#### References

* [Notes on consistent metacontent][]
* [Introduction to Augment pt. 1][]
* ["Everything is a file"][]
* [Variable][]

# Goal

Introduce a mechanism with which to associate metadata with a location in such a way that it becomes as transparent to the end-user as handling files.

Metadata is crucial and a basic component not only of computers and the systems we build, but to our psyche. Knowledge is knowledge, but so is our knowledge about this knowledge and therein lies the keyword; *about*. Meta-knowledge. Knowledge is information is `data`.

Open Metadata MUST allow for any `data` to contain metadata, including metadata itself, and it must to so in a manner that doesn't affect the original `data` (See RFC24 on "Sidecar files") and finally this data MUST NOT be bound by any particular representation; meaning it may be stored in any format capable of being represented on a file-system.

# Zen of Open Metadata

* Change is common
* Usability is more important than features
* Control is more important than performance
* Encapsulation is more important than disk space

# Architecture

Open Metadata defines two types; `location` and `entry`. Location is a reference to a folder on disk.

```python
location = '/home/marcus'
```

Entry represent and entry into a `database`; `database` is the term used for any data-store - including but not limited to relational databases, file-systems and in-memory data-structures.

An `Entry` MUST be able to contain one or more `Entry` objects.

An `Entry` that reference non-supported data-types are referred to as a `blob`, such as `jpeg` or `mp3` and are treated like incomprehensible blobs of data; usually either copied or hard-linked into a metadata repository.

Read more about Blobs in RFC16

### Data-types

A data-**format** is the physical layout of one's and zero's within the one-dimensional array of bytes that make up a file on a file-system, e.g. `jpeg`, `zip`. A data-**type** however is their interface towards the programmer - their object-type, if you will - and determines what tools are available; both textually and ultimately graphically.

All data-types native to Open Metadata MUST serialise outgoing data into JSON-compatible data and MAY deserialise incoming data using JSON as well.

Here are *all* the supported types

**Generic**

* `bool`
* `int`
* `float`
* `string`
* `enum`
* `tuple`
* `list`
* `dict`
* `null`

**Util**

* `text`
* `date`
* `color`
* `flag`
* `arg`
* `kwarg`
* `url`
* `path`

**Numbers**

* `scalar`
* `point`
* `vector`
* `matrix`

**Communication**

* `like`
* `email`
* `following`
* `message`
* `conversation`

`bool`, `int`, `float`, `string` and `date` represent simple files with an added suffix corresponding to their type, such as *myfile.string*. `enum`, `tuple` and `list` however are different from regular groups in that they are *ordered*; meaning they maintain the individual indexes of each member. This is useful when storing data that may be visualised in a UI which needs to display items in a certain order; such as a full address.


```python
# Python example
value = ['31 Quantum Tower',
    'Poland Road',
    'W21X 8SL',
    'London',
    'UK']

location = Location('/home/marcus')
entry = Entry('myaddress.list', parent=location)

entry.value = value
entry.dump()

assert entry.value == value

```

### Default values

Each entry MAY provide a default value.

* `bool` = `False`
* `int` = `0`
* `float` = `0.0`
* `string` = `''`
* `text` = `''`
* `date` = `Current Date`
* `null` = `Empty`


### Late binding

Late binding implies that functionality within an object is determined at run-time. 

In a database, each entry is stored along with a suffix that determine its type; e.g. string, float, bool etc. When assigning any value to a entry, this type is implicitly defined; similar to variables in a dynamically typed programming language such as Python:

```python
# Which data-type does `my_variable` end up with? (spoiler: a boolean)
>>> my_variable.value = 'hello'
>>> my_variable.value = 5
>>> my_variable.value = True
```

This works, because there MUST exist only one (1) suffix per entry within the database.

```python
om.write(path, '/secretOfLife, 'coconut')
om.write(path, '/secretOfLite', 47)
# This overwrites the 'secretOfLite.string' on disk
# with 'secretOfLife.int', just like it would in a
# dynamic programming language, like Python.
```

The same MAY apply to groups as well.

```python
>>> my_variable.value = 'this will make a entry of type "string"'
>>> my_variable.value = ['this will make a collection', 'of type "list"']
```

#### References

* http://en.wikipedia.org/wiki/Late_binding
* http://en.wikipedia.org/wiki/Dynamic_dispatch

### Data-formats

Native data-formats, such as `txt` or `jpeg` are treated with the minimal knowledge that their corresponding suffix allows, which in most cases are fine; a `jpeg` can only mean a rectangular bit-map with only one possible compression method.

### Location

Refer to an absolute path as *location* so as to facilitate for future expansion into using URI/URL addresses.

It MUST NOT matter to the programmer *where* the metadata is stored and it MUST NOT matter in what format that data resides. With such assumptions, we can assert valid metadata and standard use regardless of it residing on a remote file-system, within a binary file or in-memory within an application. Any content can contain metadata, regardless of what is hosting it.

### Correlation with `variable`

`Entry` is tightly connected to the terminology of a `variable` in programming languages with dynamic type-checking.

A `variable` is defined on Wikipedia as:

> a storage location and an associated symbolic name (an identifier) which contains some known or unknown quantity or information, a value. - http://en.wikipedia.org/wiki/Variable_(computer_science)

In programming languages with dynamic type-checking, the type of each variable can be modified at run-time.

```python
# For example
>>> my_var = 5
>>> my_var = 'Hello World!'
```

The same is true for the `Entry` object.

```python
>>> my_entry.value = 5
>>> my_entry.value = 'Hello World'
```

### Writing to groups

Data MAY be written directly to groups; this becomes the metadata of that collection. In the example above we dump directly to a Group object. The resulting entries are formatted according to the collection's suffix which in this case results in an ordered list.

In other cases, where the collection has no suffix, the data is formatted as-is; meaning Open Metadata will determine in which format the data is to be stored based on its object-type within the given programming language and imprint the result into the suffix of the entry.

```python

# Example of auto-determining data-type
# from suffix-less collecton of entries.
entry = Entry('mygroup', parent=location)
entry.value = ['some data']
```

This MAY introduce a possible performance penalty; due to the amount of guess-work that has to be done and so the user SHOULD explicitly specify the data-type for any given entry.

# Syntax

```python
"""Demonstration of the syntax"""

import os 
import openmetadata as om 

path = os.path.expanduser('~') 

location = Location(path) 

value = {
    'hello': 'there', 
    'startFrame': 5, 
    'endFrame': 10, 
    'hidden': True
} 

entry = Entry('keyvaluestore', parent=location)
entry.value = value
entry.dump()
  
assert entry.value == value

om.clear(location)
assert not om.find(location)
```

### Bracket-notation

Open Metadata MAY support the notion of accessing members via bracket-notation.

```python
>>> location['child']
```

The same is true for accessing list-style entries.

```python
>>> location[0]
>>> location[1]
```

### Use of `dump()`

Coupling reading and writing within the same object sure is a convenience, but also introduces a security risk. I'm not talking about someone hacking your object while you use it, but more of security for you, yourself, while using an object. Having `dump()` close to overall operation of an object, a misspelling or misuse could potentially lead to removing important information.

An alternative is to introduce a separate method responsible for dump-operations.

```python
>>> location = Location('/home/marcus')
>>> entry = Entry('description.list')
>>> entry.value = ['my', 'ordered', 'list']
>>> dump(entry)
```

It didn't take any more lines of code, yet the implementation of writing is de-coupled from the object with which the contains resides and put into a more global space from where it can be distributed appropriately if need be.

### Benefits of `dump()`

Separating objects and communication has other benefits. One of which lies in the ability to overload `dump()` with logging or networking functionality; such as wrapping up `dump()` in a custom `my_dump()` that does what `dump()` does but more; specific to your situation.

This leaves the object pure and lightweight in situations where many objects need to be used, and keeps the heavy machinery and logic on the outside, where it can more easily be scrutinised.

Another benefit is their interchangeability. `dump()` performs an action, but the action is not necessarily unique and could be replaced by another, without modifying the object, which may have become a dependency in your software already.

# Arbitrary Depth

An important aspect of Open Metadata is that of arbitrary depths; i.e. allowing for an unlimited nesting of `entry` within `entry`.

```python
+-- top folder
    +-- group1
        +-- group2
            +--group3
                +-- entry.string
```

## Mixing `entry` objects

Open Metadata MUST support the notion of mixing `entry` objects within a hierarchy.

```python
+-- top folder
    +-- group1
    +-- entry1
```

# Automatic types

Open Metadata MUST support the notion of lazily assigning data to `Entry` objects

```python
>>> location = Location('/home/marcus')

# We'll define a entry, but neglect to give it a suffix.
# The contents of this entry could be anything at this point.
>>> entry = Entry('my_simple_entry', parent=location)

# Assining data of type 'string' will automatically specify 'string' 
# as the data-type of this entry, resulting in a file on disk with
# a suffix of 'string'
>>> entry.value = 'my simple string'

>>> dump(entry)
```

# Esoteric types

In addition to what you would expect from a metadata-storage API, there is one other possibility that may keep you up at night (in a good way).

We'll cover

* `entry.stream`
* `entry.sql`
* `entry.rpc`

### `stream`

```python
+-- folder
|   +-- presentation.stream
```

It's important to remember that a file is nothing more than a logical representation of a sequence of 1s and 0s on a hard-drive. Now, whenever you stream video from YouTube, this concept is still very much in play.

It may not stream to disk, but if it did it would not matter. What matters is the sequence of 1s and 0s and how those are represented to you.

In the example above, there is a entry within a folder with the suffix 'stream'. This indicates that within this file lies instructions for how to connect to a source other than your hard-drive and to provide you with a handle to it; just like you would a regular file.

What do to with this handle however is outside of the scope of this specification and in fact outside the scope of Open Metadata itself.

What Open Metadata provides to you is the possibility of storing such an instruction in arbitrary folders on your hard-drive; it'd then be up to your front-end to interpret and possibly visualise this stream for you.

### `sql`

Any database is ultimately just one or more files on some disk. You could gain access to this file, but it would bear little meaning. What would be more interesting however is to attain a handle into a particular portion of an SQL-based database and manipulate it just like you would a regular Open Metadata entry. Perhaps even store this handle somewhere in your local hard-drive, as metadata to a folder.

```python
>>> location = Location('/some/folder')
>>> location.tree()
# +-- folder
# |   +-- startFrame.sql
```

### `rpc`

How about reading and writing data via a remote procedure call (RPC)? The entry could contain instructions for either and get interpreted by your application.

```python
>>> location = Location('/some/folder')
>>> location.tree()
# +-- folder
# |   +-- startFrame.rpc
```


[Variable]: http://en.wikipedia.org/wiki/Variable_(computer_science)
[Pipi]: http://pipi.io
["Everything is a file"]: http://www.abstractfactory.io/blog/everything-is-a-file/
[Introduction to Augment pt. 1]: http://www.abstractfactory.io/blog/introduction-to-augment-pt-1/
[Notes on consistent metacontent]: http://www.abstractfactory.io/blog/notes-on-consistent-metacontent/