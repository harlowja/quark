=====
Quark
=====

Mission
-------

To be a scalable compute(r) maker & manipulator (in a box).

What it is
----------

* **Simple** enabler of stated mission.
* **Scalable** enabler of stated mission.
* A project that you can **enjoy** working
  with (and operating).

What it is not
--------------

* Meant to be complex.
* A toaster oven.
* A pile of crap.

How it's meant to work
----------------------

Basics
******

There are three parts of ``quark``, the first being a ``quark-compute``
which is the actual integration point with ``libvirt``; this component is
actively placing information about itself into a zookeeper directory that
it is responsible for allocating (there would be assumed to be many
hundred or thousands of these ``quark-compute`` processes and directories).

A sample/example of that directory is the following::

    self/ (used for a external entity to select a compute)
       - ram
       - cpu
       - network/
            - max_bandwith
       ...
    active_tasks/
      - 4058b545-de95-4b3c-aed9-dfd0e5945001/
          - kind (enum, build, delete, ...)
          - birthed_on
          - steps_completed/
              - allocate_vcpu
                  - time_to_completion
          - steps_left/
              - allocate_storage
              - start
          - consumes/
              - ram
              - cpu
              - ...
    managing/
       - ba5902e9-e20b-42df-bafd-34e98a846ddb
           - kind
           - birthed_on
           - consumes/
              - ram
              - cpu
              - ...
           - status
    next_tasks/
       - 6d3ed5e1-3ad3-4dae-8928-32ef5a38008e
           - kind
           - consumes/
              - ram
              - cpu
              - ...

The job of ``quark-compute`` would be to update the above data structures
and shift the ``next_tasks`` (either internally or externally requested
work) to ``active_tasks`` and ensure that the ``self`` directory has a
accurate view.

The second key component of quark is ``quark-api`` which is the REST
endpoint of quark, it is what users will be calling into so that they
can initiate some kind of compute creation or manipulation. It will do
very little work itself and will primarily be responsible for producing
a ``work-unit-id`` that the user will recieve that they can then poll on to
get status about there requested work they had desired to be completed.

The final component (and one of the more complex) is ``quark-orc`` which
is the component in ``quark`` that is meant to orchestrate (and/or initiate)
complex requests that span across the cluster. For example, it will be
the ``quark-orc`` which will be having watches setup on all
the ``quark-compute`` directories (so that the ``quark-orc`` can be aware
of the changes those ``quark-compute`` nodes are undergoing). It will be
the ``quark-orc`` job to recieve work requests (from the ``quark-api``
and drive them to completion); for example to boot a new virtual machine
the ``quark-orc`` would need to go through a set of tasks that examine its
in-memory version of the cluster and determine where a new ``next_task``
should be placed.

The ``quark-orc`` will be a distributed component (that itself will have
a similar zookeeper directory to what the ``quark-compute`` directory
structure is); although it will be slightly different as the ``quark-orc``
is more task driven (and likely does not need a ``self`` concept).

Why we made it
--------------

::

    Spoon boy: Do not try and bend the spoon. That's impossible.
               Instead... only try to realize the truth.
    Neo: What truth?
    Spoon boy: There is no spoon.
    Neo: There is no spoon?
    Spoon boy: Then you'll see, that it is not the spoon that
               bends, it is only yourself.
