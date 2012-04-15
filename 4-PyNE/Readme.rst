PyNE: Python for Nuclear Engineering
____________________________________________


`Back To Python <https://github.com/thehackerwithin/physor2012/tree/master/3d-PythonFunctionsAndModules>`_ - 
`Forward To Testing <http://github.com/thehackerwithin/UofCSCBC2012/tree/master/5-Testing/>`_

**Presented and Designed by Anthony Scopatz** 

Motivation
================================
`PyNE`_, or Python for Nuclear Engineering, is a suite of tools to aid in 
computational nuclear science & engineering.  PyNE seeks to provide 
native implementations of common nuclear algorithms, as well as Python
bindings and I/O support for other industry standard nuclear codes.

PyNE came out of discussions between Seth Johnson, Paul Romano, and 
Anthony Scopatz about what a common nuclear engineering tool suite is,
where it should live, and what restrictions should be placed on it.
To date we have seen a year of development - mostly incorporating 
pre-existing features.

At it's core is a Python-independent C++ library.  There is then a 
wrapper layer (written in Cython) which exposes the C++ functionality 
back to Python.  This architechure makes PyNE both fast and usable.

.. _PyNE: http://pyne.github.com/


Current Contents
==================================
PyNE currently has the following sub-packages:

* Nuclide Naming Module - pyne.nucname
* Basic Nuclear Data - pyne.data
* Materials - pyne.material
* Binning Functions - pyne.bins
* CCCC Formats - pyne.cccc
* ACE Cross Sections - pyne.ace
* ENSDF File Support - pyne.ensdf
* Cross Section Interface - pyne.xs
* ORIGEN 2.2 Support - pyne.origen22
* Serpent Support - pyne.serpent
* Utility Functions - pyne.utils

There are also some back-end utilities for interfacing with C++ and
for building a database of basic nuclear information.

Nuclide Naming Conventions
=========================================
One of the most frustrating aspects of nuclear data software is the large number
of different ways that people choose to nuclide names.  Functionally, there are 
three pieces of information that *should* be included in a radionuclide's name

1. **Z Number**: The number of protons.
2. **A Number**: The number of nucleons (neutrons + protons).
3. **Excitation Level**: The internal energy state of the nucleus.

Some common naming conventions exist.  The following are the ones currently 
supported by PyNE.

 #. **zzaaam**: This type places the charge of the nucleus out front, then has three
    digits for the atomic mass number, and ends with a metastable flag (0 = ground,
    1 = first excited state, 2 = second excited state, etc).  Uranium-235 here would
    be expressed as '922350'.
 #. **name**: This is the more common, human readable notation.  The chemical symbol
    (one or two characters long) is first, followed by the atomic weight.  Lastly if
    the nuclide is metastable, the letter *M* is concatenated to the end.  For example,
    'H-1' and 'Am242M' are both valid.  Note that nucname will always return name form with
    the dash removed and all letters uppercase.
 #. **MCNP**: The MCNP format for entering nuclides is unfortunately non-standard.  In most
    ways it is similar to zzaaam form, except that it lacks the metastable flag.  For information
    on how metastable isotopes are named, please consult the MCNPX documentation for more information.
 #. **Serpent**: The serpent naming convention is similar to name form.  However, only the first
    letter in the chemical symbol is uppercase, the dash is always present, and the the meta-stable
    flag is lowercase.  For instance, 'Am-242m' is the valid serpent notation for this nuclide.
 #. **NIST**: The NIST naming convention is also similar to the Serpent form.  However, this
    convention contains not meta-stable information.  Moreover, the A-number comes before the
    element symbol.  For example, '242Am' is the valid NIST notation.
 #. **CINDER**: The CINDER format is similar to zzaaam form except that the placement of the Z- and
    A-numbers are swapped. Therefore, this format is effectively aaazzzm.  For example, '2420951' is
    the valid cinder notation for 'AM242M'.

Canonical Form
--------------
The ``zzaaam`` integer form of nuclide names is the fundamental form of nuclide naming because
it accurately captures all of the needed information in the smallest amount of space.  
Given that the Z-number may be up to three digits, A-numbers are always three digits, and 
the excitation level is one digit, all possible nuclides are represented on the range 
``0 <= zzaaam < 10000000``.  This falls well within 32 bit integers.

On the other hand, ``name`` string representations may be anywhere from two characters (16 bits)
up to six characters (48 bits).  So in general, ``zzaaam`` is smaller by 50%.  

The other distinct advantage that integer forms have is that you can natively perform arithmetic
on them.  For example:

.. code-block:: python

    # Am-242m
    nuc = 942421

    # Z-number
    zz = nuc/10000

    # A-number
    aaa = (nuc/10)%1000

    # Meta-stable state
    m = nuc%10

Code internal to PyNE uses ``zzaaam``, and except for human readability, you should too!  
Natural elements are specified in this form by having zero A-number and excitation flags
(``zzaaam('U') == 920000``).

Examples of Use
---------------

.. code-block:: python

    In [1]: from pyne import nucname

    In [2]: nucname.zzaaam('U235')
    Out[2]: 922350

    In [3]: nucname.zzaaam(10010)
    Out[3]: 10010

    In [4]: nucname.name(10010)
    Out[4]: 'H1'

    In [5]: nucname.serpent('AM242M')
    Out[5]: 'Am-242m'

    In [6]: nucname.name_zz['SR']
    Out[6]: 38

    In [7]: nucname.zz_name[57]
    Out[7]: 'LA'


Materials
===============================
PyNE contains a Material class, which is used to represent nuclear
materials.  All functionality may be found in the ``material`` package:

.. code-block:: python

    from pyne.material import Material

Materials are the primary container for radionuclides. They map nuclides to **mass weights**,
though they contain methods for converting to/from atom fractions as well.
In many ways they take inspiration from numpy arrays and python dictionaries.  Materials
have two main attributes which define them.

1. **comp**: a normalized composition mapping from nuclides (zzaaam-ints) to mass-weights (floats).
2. **mass**: the mass of the material.

By keeping the mass and the composition separate, operations that only affect one attribute
may be performed independent of the other.  Additionally, most of the functionality is
implemented in a C++ class by the same name, so this interface is very fast and light-weight.
Materials may be initialized in a number of different ways.  For example, initializing from
dictionaries of compositions are shown below.

.. code-block:: python

    In [1]: from pyne.material import Material

    In [2]: leu = Material({'U238': 0.96, 'U235': 0.04}, 42, 'LEU')

    In [3]: leu
    Out[3]: pyne.material.Material({922350: 0.04, 922380: 0.96}, 42.0, 'LEU', -1.0)

    In [4]: nucvec = {10010:  1.0, 80160:  1.0, 691690: 1.0, 922350: 1.0,
       ...:           922380: 1.0, 942390: 1.0, 942410: 1.0, 952420: 1.0,
       ...:           962440: 1.0}

    In [5]: mat = Material(nucvec)

    In [6]: print mat
    Material: 
    mass = 9.0
    atoms per molecule = -1.0
    -------------------------
    H1     0.111111111111
    O16    0.111111111111
    TM169  0.111111111111
    U235   0.111111111111
    U238   0.111111111111
    PU239  0.111111111111
    PU241  0.111111111111
    AM242  0.111111111111
    CM244  0.111111111111

Materials may also be initialized from plain text or HDF5 files (see ``Material.from_text()`` and
``Material.from_hdf5()``).  


Material Arithmetic
-------------------
Furthermore, various arithmetic operations between Materials and numeric types are also defined.
Adding two Materials together will return a new Material whose values are the weighted union
of the two original. Multiplying a Material by 2, however, will simply double the mass.

.. code-block:: python

    In [11]: other_mat = mat * 2

    In [12]: other_mat
    Out[12]: pyne.material.Material({10010: 0.111111111111, 80160: 0.111111111111, 691690: 0.111111111111, 
       ...:                          922350: 0.111111111111, 922380: 0.111111111111, 942390: 0.111111111111, 
       ...:                          942410: 0.111111111111, 952420: 0.111111111111, 962440: 0.111111111111}, 
       ...:                          2.0, '')

    In [13]: other_mat.mass
    Out[13]: 2.0

    In [14]: weird_mat = leu + mat * 18

    In [15]: print weird_mat
    Material: 
    mass = 60.0
    atoms per molecule = -1.0
    -------------------------
    H1     0.0333333333333
    O16    0.0333333333333
    TM169  0.0333333333333
    U235   0.0613333333333
    U238   0.705333333333
    PU239  0.0333333333333
    PU241  0.0333333333333
    AM242  0.0333333333333
    CM244  0.0333333333333


Indexing & Slicing
------------------
Additionally (and very powerfully!), you may index into either the material or the composition 
to get, set, or remove sub-materials.  Generally speaking, the composition you may only index 
into by integer-key and only to retrieve the normalized value.  Indexing into the material allows 
the full range of operations and returns the unnormalized mass weight.  Moreover, indexing into
the material may be performed with integer-keys, string-keys, slices, or sequences of nuclides.

.. code-block:: python

    In [22]: leu.comp[922350]
    Out[22]: 0.04

    In [23]: leu['U235']
    Out[23]: 1.68

    In [24]: weird_mat['U':'Am']
    Out[24]: pyne.material.Material({922350: 0.0736, 922380: 0.8464, 942390: 0.04, 942410: 0.04}, 50.0, '', -1.0)

    In [25]: other_mat[:920000] = 42.0

    In [26]: print other_mat
    Material: Other Material
    mass = 50.3333333333
    atoms per molecule = -1.0
    -------------------------
    H2     0.834437086093
    U235   0.165562913907

    In [27]: del mat[962440, 'TM169', 'Zr90', 80160]

    In [28]: mat[:]
    Out[28]: pyne.material.Material({10010: 0.166666666667, 922350: 0.166666666667, 922380: 0.166666666667, 
       ...:                          942390: 0.166666666667, 942410: 0.166666666667, 952420: 0.166666666667}, 
       ...:                          0.666666666667, '', -1.0)

Other methods also exist for obtaining commonly used sub-materials, such as gathering the Uranium 
or Plutonium vector.  


Molecular Weights & Atom Fractions
----------------------------------
You may calculate the molecular weight of a material via the ``Material.molecular_weight()`` 
method. 

.. code-block:: python

    In [29]: leu.molecular_weight()
    Out[29]: 237.9290388038301

Note that by default, materials are assumed to have one atom per molecule.  This is a poor
assumption for more complex materials.  For example, take water.  Without specifying the 
number of atoms per molecule, the molecular weight calculation will be off by a factor of 3.
This can be remedied by passing the correct number to the method.  If there is no other valid
number of molecules stored on the material, this will set the appropriate attribute on the 
class.

.. code-block:: python

    In [30]: h2o = Material({10010: 0.11191487328808077, 80160: 0.8880851267119192})

    In [31]: h2o.molecular_weight()
    Out[31]: 6.003521561343334

    In [32]: h2o.molecular_weight(3.0)
    Out[32]: 18.01056468403

    In [33]: h2o.atoms_per_mol
    Out[33]: 3.0

It is often also useful to be able to convert the current mass-weighted material to 
an atom fraction mapping.  This can be easily done via the ``Material.to_atom_frac()``
method.  Continuing with the water example, if the number of atoms per molecule is 
properly set then the atom fraction return is normalized to this amount.  Alternatively, 
if the atoms per molecule are set to its default state on the class, then a truly 
fractional number of atoms is returned.

.. code-block:: python

    In [34]: h2o.to_atom_frac()
    Out[34]: {10010: 2.0, 80160: 1.0}

    In [35]: h2o.atoms_per_mol = -1.0

    In [36]: h2o.to_atom_frac()
    Out[36]: {10010: 0.666666666667, 80160: 0.333333333333}

Additionally, you may wish to convert the an existing set of atom fractions to a 
new material stream.  This can be done with the :meth:`Material.from_atom_frac` method, 
which will clear out the current contents of the material's composition and replace
it with the mass-weighted values.  Note that 
when you initialize a material from atom fractions, the sum of all of the atom fractions
will be stored as the atoms per molecule on this class.  Additionally, if a mass is not 
already set on the material, the molecular weight will be used.

.. code-block:: python

    In [37]: h2o_atoms = {10010: 2.0, 'O16': 1.0}

    In [39]: h2o = from_atom_frac(h2o_atoms)

    In [40]: h2o.comp
    Out[40]: {10010: 0.111914873288, 80160: 0.888085126712}

    In [41]: h2o.atoms_per_mol
    Out[41]: 3.0

    In [42]: h2o.mass
    Out[42]: 18.01056468403

    In [43]: h2o.molecular_weight()
    Out[43]: 18.01056468403

Moreover, other materials may also be used to specify a new material from atom fractions.
This is a typical case for reactors where the fuel vector is convolved inside of another 
chemical form.  Below is an example of obtaining the Uranium-Oxide material from Oxygen
and low-enriched uranium.

.. code-block:: python

    In [44]: uox = Material()

    In [45]: uox.name = "UOX"

    In [46]: uox.from_atom_frac({leu: 1.0, 'O16': 2.0})

    In [47]: print uox
    Material: UOX
    mass = 269.918868043
    atoms per molecule = 3.0
    ------------------------
    O16    0.118516461895
    U235   0.0352593415242
    U238   0.846224196581

.. note:: Materials may be used as keys in a dictionary because they are hashable.

Further information on the Material class may be seen in the library reference :ref:`pyne_material`.

