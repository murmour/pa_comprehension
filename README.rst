================
Pa_comprehension
================

Pa_comprehension implements the comprehension expression construct,
using the Enum data structure from Batteries for the underlying machinery.

See INSTALL.txt for building and installation instructions.
See LICENSE.txt for copying conditions.

Home page: https://github.com/cakeplus/pa_comprehension


Syntax description
==================

The global form is::

  [? output | comp_item ; comp_item ; ... ?]

A ``comp_item`` is either a guard (a boolean expression), or a generator of the
form ``<patt> <- <expr>``. Variables bound in the pattern can be used in the
following ``comp_items``, and in the output expression.


Module parametrization
======================

Both the output and the generator expression can be optionally prefixed with
a module specifier:

.. sourcecode:: ocaml

  [? Module: output | ... ?]  and  patt <- Module: expr

In the output position, it specifies the data structure module (e.g. List) used
for the whole comprehension value. In the generator position, it specifies the
data structure module corresponding to the generator expression, e.g.:

.. sourcecode:: ocaml

  p <- List: [1; 2; 3]

In the absence of module specifier, the Enum module from Batteries is used.

Comprehension expressions rely on the presence of the following operations
in the given module (where 'a t represents the data-structure type):

.. sourcecode:: ocaml

   val filter     : ('a -> bool) -> 'a t -> 'a t
   val concat     : 'a t t -> 'a t
   val map        : ('a -> 'b) -> 'a t -> 'b t
   val filter_map : ('a -> 'b option) -> 'a t -> 'b t
   val enum       : 'a t -> 'a Enum.t
   val of_enum    : 'a Enum.t -> 'a t

If your module does not provide the first four operations but only the Enum
conversion functions, you could still benefit from the comprehension syntax
by using ``foo <- Mod.enum bar`` instead of ``foo <- Mod: bar``.


Example
=======

Input:

.. sourcecode:: ocaml

  [? Array: pow p k | p <- 2--100; prime p; k <- List: (seq 1 (p - 1)) ?]

Output:

.. sourcecode:: ocaml

  Array.of_enum
   (Enum.concat
     (Enum.map
       (fun p -> Enum.map (fun k -> pow p k) (List.enum (seq 1 (p - 1))))
       (Enum.filter (fun p -> prime p) (2 -- 100))))
