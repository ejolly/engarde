Engarde
=======

[![Build Status](https://travis-ci.org/ejolly/engarde.svg)](https://travis-ci.org/ejolly/engarde)  

A python package for defensive data analysis.
Documentation is at [readthedocs](http://engarde.readthedocs.org/en/latest/).  

This is a fork of Tom Augspurger's [original engarde library](https://github.com/engarde-dev/engarde), which is **no longer maintained** and has been ported to [bulwark](https://github.com/zaxr/bulwark). I prefer working from this project because the code base is lighter, simpler, and custom checks/decorators return _booleans_ as oppose to _assertion statements_. Asserts are preferred because they are more extensible and pythonic, but if your intend to keep your checks small then returning booleans almost always results in writing less code.

- [Dependencies](#dependencies)
- [Why](#why)
- [Examples](#examples)
- [New Additions](#new-additions)

Dependencies
============

- pandas

Supports python ~~2.7+~~ and 3.4+

Why?
====

Data are messy.
But, our analysis often depends on certain assumptions about our data
that *should* be invariant across updates to your dataset.
`engarde` is a lightweight way to explicitly state your assumptions
and check that they're *actually* true.

This is especially important when working with flat files like CSV
that aren't bound for a more structured destination (e.g. SQL or HDF5).

Examples
========

There are two main ways of using the library, which correspond to the
two main ways I use pandas: writing small scripts or interactively at
the interpreter.

First, as decorators, which are most useful in `.py` scripts
The basic idea is to  write each step of your ETL process as a function
that takes and returns a DataFrame. These functions can be decorated with
the invariants that should be true at that step in the process.

```python
from engarde.decorators import none_missing, unique_index, is_shape

@none_missing()
def f(df1, df2):
    return df1.add(df2)

@is_shape((1290, 10))
@unique_index
def make_design_matrix('data.csv'):
    out = ...
    return out
```

Second, interactively.
The cleanest way to integrate this is through the [``pipe``](http://pandas-docs.github.io/pandas-docs-travis/basics.html#tablewise-function-application) method,
introduced in pandas 0.16.2 (June 2015).

```python
>>> import engarde.checks as dc
>>> (df1.reindex_like(df2)
...     .pipe(dc.unique_index)
...     .cumsum()
...     .pipe(dc.within_range, (0, 100))
... )
```

New Additions
=============

Like the core library, these additions can be imported and used as either functions or decorators.  

- `reset_index`
    - Check if a dataframe has a numeric index equal to 0 - nrows-1
- `grps_have_same_nunique_val(grpcol, valcol)`
    - Check if `df.groupby(grpcol)[valcol.nunique()` is the same for each group
- `grps_have_same_nobs(grpcol, nobs=None)`
    - Check if `df.groupby(grpcol).size()` is the same for each group and optional that all groups have `nobs`
