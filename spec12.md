# Object-Oriented Metadata (OOM)

An extension to Open Metadata to support the notion of inheritance.

![](https://dl.dropbox.com/s/2eyg655o4cws97t/oom_place_v001.png)

* Name: https://github.com/abstract-factory/rfc/spec:12 (12/OOM)
* Editor: Marcus Ottosson <marcus@abstractfactory.io>
* Inherits: RFC1
* State: draft

Copyright, Change Process and Language is derived via inheritance as per [RFC1][].

# Goal

Open Metadata is a method for storing generic data within folders on your hard-drive. This document describes a method that defines each folder as an object within an inheritance-tree; each folder inheriting from its parent.

The result of this is a cascading behaviour in attributes from top to bottom within a hierarchy.

For motivation and use of cascading data, head over to Steve Yegge's inspiration post about the [Universal Design Pattern][]

### When to use

When is inheritance in information intuitive? Well, consider configuration files of software. Consider Sublime Text. In Sublime, you have Configuration Files (CF) and then you have User Configuration Files (UCF). The information within UCF augments and, perhaps more importantly, overrides the CF. You may copy certain information over from the CF into the UCF in order to make alterations and you may add your own configuration that would augment the original configuration.

### When not to use

Inheritance is not suited to all types of information. Consider the description of the folder "my mp3 collection". This description is singular and does not apply to any descenants in a hierarchy; like "best of bon jovi".

# Architecture

OOM extends Open Metadata with the `Instance` and `Attribute` objects and an `inherit()` method. Open Metadata defines `om.pull()` as a means of reading from disk; `om.inherit()` pulls data from both current and ascending folders. `Instance` then wraps `om.Location` to replicate the interface of a regular plain-data-type programming object.

```python
>>> config = oom.Instance(r'c:\users\marcus\sublime')
>>> config.plugins = ['spell_correction']
```

With `oom.inherit()`, each folder may be thought of as a subclass of its parent, passing on and potentially overriding information from parent to child; just as you would expect from regular inheritance in Object-oriented languages such as Python or C++.

```python
>>> userconfig = oom.Instance(r'c:\users\marcus\sublime\user')
>>> oom.inherit(userconfig)
>>> userconfig.plugins = ['my_custom_plugin']
>>> userconfig.plugins
['spell_correction', 'my_custom_plugin']
```

## [Liskov Substitution Principle][]

Inheritance is additive. It may be easier to think of inheritance as being cascading. Cascading inheritance enforces the LSP in that however you edit children, a parent may always be replaced by its child.

This is an important concept in a progammable hierarchy and one that is enforced in the design of OOM. If a child were to be given the ability to break the LSP, then you could never be sure at which point in your hierarchy your code will fail.

```python
>>> oom.remove(userconfig.plugins)
```

## Difference between `Location` and `Instance`

`Instance` is essentially a `Location`, with a few minor differences.

1. `Location` does not take into account ascending parent-hood.
2. `Location` cannot have its members accessed via dot-notation.
3. and thus `Location` cannot have its members modified directly.

## Designed for inheritance

In the above example, how would you go about *removing* or *altering* inherited information? Consider the following example

```python
>>> project = oom.Instance(r'/project')
>>> project.executables
['maya', 'houdini']
```

Underneath 'project' we have 'shot5', but we don't want 'maya' to be part of 'shot5', so what do we do?

```python
>>> project = oom.Instance('/project')
>>> project.executables
{'maya': True, 'houdini': True}
```

Now both 'maya' and 'houdini' have a bool value specifying whether or not they are active. Inactivating any executable is now a matter of:

```python
>>> shot = oom.Instance('/project/shot5')
>>> shot.executables = {'maya': False}
>>> shot.executables
{'maya': False, 'houdini': True}
```

### Selective inheritance

What about situations where you aren't looking to inherit *every* member of an instance? This is the recommended way with which to retrieve metadata in an inheritance manner as it greatly decreases the amount of reading done in each query.

```python
>>> shot = oom.Instance('/project/shot5')
>>> oom.inherit(shot.executables)
>>> shot.executables
{'maya': False, 'houdini': True}
```

### Shallow inheritance

How about situations where you may only be interested in an immediate parent, and not any of the parents above?

```python
>>> oom.pull(oom.Instance('/')).data == ['a']
>>> oom.pull(oom.Instance('/project')).data == ['b']
>>>
>>> shot = oom.Instance('/project/shot5')
>>> oom.inherit(shot, level=1)
>>> shot.data = 'c'
>>> shot.data
['b', 'c']
```

# Implementation

```python
# Psuedo-code
class Instance:
	def __init__:
		children = []
		inherited_children = []

	def data:
		return inherited_children + children

```

Each Instance contain a `inherited_children` member which is mostly empty when not utilising `oom.inherit()`

After `oom.inherit()` `inherited_children` is populated with.. inherited children (i.e. data) that is added upon query, but not altered upon edit.

```python
# This will add '5' to 'instance', but won't remove inherited data
>>> instance.data
['hello']
>>> instance.data = [5]
>>> instance.data
['hello', 5]
```

### Inheritance by either Instance or Attribute

```python
def inherit_instance()
   pass

def inherit_attribute()
   pass

def inherit(item)
   if isinstance(item, Instance):
      return inherit_instance(item)
   elif isinstance(item, Attribute):
      return inherit_attribute(item)
   raise TypeError
```

[Liskov Substitution Principle]: http://en.wikipedia.org/wiki/Liskov_substitution_principle
[universal design pattern]: http://steve-yegge.blogspot.co.uk/2008/10/universal-design-pattern.html