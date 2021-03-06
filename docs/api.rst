.. _api:

API
===

.. testsetup:: *

   from liblet import Production, Item, closure, show_calls, Transition


Productions, Grammars and Derivations
-------------------------------------

.. currentmodule:: liblet.grammar

The basic building block of a *grammar* is a *production* represented by the following class.

.. autoclass:: Production

   A convenience method is provided to obtain productions from a suitable string
   representation of them.
   
   .. automethod:: from_string
   
   In order to find productions satisfying a set of properties, a convenience method 
   to build a predicate from a set of keyword arguments is provided.  

   .. automethod:: such_that

For the purpose of presenting the `Knuth Automaton <https://en.wikipedia.org/wiki/LR_parser#Finite_state_machine>`__ 
in the context of LR(0) parsing,  productions are extended to include a dot in the following class.

.. autoclass:: Item

A grammar can be represented by the following class, that can be instantiated
given the usual formal definition of a grammar as a tuple.

.. autoclass:: Grammar

   Even in the case of grammars, a convenience method is provided to build a
   grammar from a suitable string representation. Once the productions are obtained
   with :func:`Production.from_string`, the remaining parameters are
   conventionally inferred as detailed below.
   
   .. automethod:: from_string
   
   An utility method is provided to enumerate alternatives for a given left-hand side:
   
   .. automethod:: alternatives

   To implement grammar "cleanup strategies" a method to eliminate unwanted symbols (and productions)
   from a grammar is also provided.

   .. automethod:: restrict_to

Once a grammar has been defined, one can build derivations with the help of the
following class.

.. autoclass:: Derivation

   .. automethod:: step
   
   .. automethod:: leftmost

   .. automethod:: rightmost

   .. automethod:: possible_steps

   .. automethod:: steps

   .. automethod:: sentential_form

Derivations can be displayed using a :obj:`~liblet.display.ProductionGraph`.

Transitions and Automata
------------------------

.. currentmodule:: liblet.automaton

Albeit in the context of parsing (that is the main focus of this library), the
role of *finite state automata* and *regular grammars* is not the main focus, a
couple of classes to handle them is provided.

First of all, a basic implementation of a *transition* is provided; as in the
case of grammars, the symbols and terminals are simply :obj:`strings <str>`.

.. autoclass:: Transition

   A convenience method is provided to obtain transitions from a suitable string
   representation of them.

   .. automethod:: from_string


From transitions one can obtain a representation of a (*nondeterministic*)
*finite state automata* using the following class.

.. autoclass:: Automaton

   Even for the automata there are a couple of convenience methods to obtain
   automata from a string representation of the transitions, or from a *regular
   grammar*.

   .. automethod:: from_string

   .. automethod:: from_grammar

    An utility method is provided to represent the *transition function* of the automata.

   .. automethod:: δ

Automata can be displayed using :obj:`StateTransitionGraph.from_automaton <liblet.display.StateTransitionGraph.from_automaton>`.

ANTLR support
-------------

This library provides a commodity class to deal with `ANTLR <https://www.antlr.org/>`_ for the Python 3 *target*.

.. currentmodule:: liblet.antlr

.. autoclass:: ANTLR

   .. automethod:: print_grammar
   .. automethod:: context
   .. automethod:: tokens
   .. automethod:: tree

.. autoclass:: AnnotatedTreeWalker

   .. automethod:: catchall
   .. automethod:: register
   .. automethod:: RECOURSE_CHILDREN
   .. automethod:: TREE_CATCHALL
   .. automethod:: TEXT_CATCHALL


Rich display
------------

.. currentmodule:: liblet.display

.. autoclass:: Graph

   .. automethod:: from_adjdict

.. autoclass:: Tree

   .. automethod:: from_lol
   .. automethod:: with_threads

.. autoclass:: ProductionGraph

.. autoclass:: StateTransitionGraph

   .. automethod:: from_automaton


Utilities and decorators
------------------------

.. currentmodule:: liblet.utils

.. autoclass:: Queue
.. autoclass:: Stack
.. autofunction:: warn
.. autofunction:: peek
.. autofunction:: union_of

.. currentmodule:: liblet.decorators

.. autofunction:: closure 

.. autofunction:: show_calls