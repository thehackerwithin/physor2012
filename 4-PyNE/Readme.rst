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


