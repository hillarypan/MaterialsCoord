# MaterialsCoord

MaterialsCoord provides an infrastructure for comparing coordination numbers produced by different
approaches against a benchmark set of structures.

##### Table of Contents
[1. Why is MaterialsCoord useful](#why_is_materials_coord_useful)

[2. How do I benchmark Coordination Number algorithms with MaterialsCoord?](#how_do_i_benchmark_cn_algos)

[3. How do I implement a new Coordination Number algorithm in MaterialsCoord?](#how_do_i_implement_cn_algos)

[4. How can I use MaterialsCoord on my own structures?](#how_can_i_use_my_own_structures)

<a name="why_is_materials_coord_useful"/>
## Why is MaterialsCoord useful?

Atomic coordination numbers (CNs) are one of the most important descriptors for the local environments in condensed materials, but they
have no universal definition. Therefore, researchers have come up with a wide range of algorithms, performance of which are often
unknown outside the chemical domain they were particularly designed for. MaterialsCoord aims at providing the necessary infrastructure
to host, compare and benchmark different CN algorithms against a selected set of crystal structures. It further allows
human interpretations of CN environments to be incorporated into benchmarking (in progress).

<a name="how_do_i_benchmark_cn_algos"/>
## How do I benchmark Coordination Number algorithms with MaterialsCoord?

Examples of how MaterialsCoord benchmarking is performed can be found in [tests.ipynb](https://github.com/aykol/MaterialsCoord/blob/master/tests.ipynb)
file.

<a name="how_do_i_implement_cn_algos"/>
## How do I implement a new Coordination Number algorithm in MaterialsCoord?

This is fairly simple:

1. Define a new class that is subclassed from materialscoord.core.CNBase.
2. The class must have a method named `compute` which takes a pymatgen Structure and site-index as input,
and returns a dictionary of CNs for that site; e.g. `{'O': 4.4, 'F': 2.1}`.

For example:
```python
class MyCoordinationNumberAlgorithm(CNBase):
    """
    My new algorithm
    """
    def compute(self, structure, n):
        params = self._params

        # ... here your algorithm finds CNs of site n.
        # e.g. cns = my_algorithm(structure, n, **params)

        return cns
```

Any parameters the algorithm needs can be passed to the class when initializing using the params keyword as a dictionary. These can later
be accessed from `compute` method as _params property of the class.

In method `compute` you can do whatever is necessary to interface the algorithm with MaterialsCoord. Options include:
* Simplest: Implementing the entire algorithm within the `compute` method.
* Recommended: Add the necessary "bulky" code to a relevant module in external_src package and import as you define
  `compute`.
* Not recommended: call or import a library/program outside of MaterialsCoord within the `compute` method.
This is not recommended as external dependencies will restrict portability,
but maybe unavoidable if the external algorithm is part of another python package,
or is written in some other language such as Java. In that case `compute` can simply serve as a wrapper that calls
and post processes the output of the external code.

<a name="how_can_i_use_my_own_structures"/>
## How can I use MaterialsCoord on my own structures?
There are different ways of how one can do this.
* Add a new "group" folder that includes your structures into the test_structures folder. Then you can initialize Benchmark with
your `structure_groups = group_name` (i.e. the name of your folder). You can then call your new structure group any time you want.
* If you don't want to permanently add the structures to MaterialsCoord but rather run CN methods on some external structures,
you can use the `custom_set` argument of `Benchmark` to provide the path to a set of structure files. If `custom_set` is given,
`Benchmark` will ignore any `structure_group` provided.

Note that in any case, the structures provided can be of any type that pymatgen can automatically interpret (cif, POSCAR, etc.)