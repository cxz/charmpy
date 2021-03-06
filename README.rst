=======
CharmPy
=======

CharmPy is a general-purpose parallel and distributed programming framework with a
simple and powerful API, based on migratable Python objects and remote method
invocation; built on top of an adaptive C++ runtime system providing *speed*,
*scalability* and *dynamic load balancing*.

CharmPy allows development of parallel applications that scale from laptops to
supercomputers, using the Python language.

Please see the Documentation_.

Short Example
-------------

The following computes Pi in parallel, using any number of machines and processors:

.. code-block:: python

    from charmpy import *
    from math import pi
    import time

    class Worker(Chare):

        def work(self, n_steps, pi_future):
            h = 1.0 / n_steps
            s = 0.0
            for i in range(self.thisIndex, n_steps, charm.numPes()):
                x = h * (i + 0.5)
                s += 4.0 / (1.0 + x**2)
            # perform a reduction among members of the group, sending the result to the future
            self.contribute(s * h, Reducer.sum, pi_future)

    def main(args):
        n_steps = 1000
        if len(args) > 1:
            n_steps = int(args[1])
        mypi = charm.createFuture()
        workers = Group(Worker)  # create one instance of Worker on every processor
        t0 = time.time()
        workers.work(n_steps, mypi)  # invoke 'work' method on every worker
        print('Approximated value of pi is:', mypi.get(),  # 'get' blocks until result arrives
              'Error is', abs(mypi.get() - pi), 'Elapsed time=', time.time() - t0)
        charm.exit()

    charm.start(main)


This is a simple example and demonstrates only a few features of CharmPy. Some things to note
from this example:

- *Chares* are distributed Python objects.
- A *Group* is a type of distributed collection where one instance of the specified
  chare type is created on each processor.
- Remote method invocation in CharmPy is *asynchronous*.

In this example, there is only one chare per processor, but multiple chares (of the same
or different type) can exist on any given processor, which can bring performance
benefits. Please refer to the documentation_ for more information.


Contact
-------

`Mailing list`_

Main author at <jjgalvez@illinois.edu>



.. _Documentation: https://charmpy.readthedocs.io

.. _Mailing list: charm@cs.illinois.edu
