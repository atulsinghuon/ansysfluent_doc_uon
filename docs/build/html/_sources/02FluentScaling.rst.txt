Some tips and tricks to improve scaling
=======================================

Calculate number of cores required for Fluent job
-------------------------------------------------

This section deals with figuring out the number for the --ntasks flag, which may
vary as per the user's requirement, and the capability of a partition. This
becomes crucial, as one of the primary objectives of utilizing a High-
Performance Computing (HPC) facility is to enhance performance by
minimizing job execution time. Selecting incorrect values for --ntasks and
nodes when submitting a job can lead to either a resource error or an
escalation in communication overhead. This, in turn, extends the job's
execution time, undermining the efficiency of the computation.

An important factor to note is that Ansys recommends using 200,000 cells/core
(for research licenses) and 100,000 cells/core (for student licenses) for parallel
runs. This is a safe estimate, although users are welcome to calculate by
tweaking this number as this is application-dependent. For instance,
combustion and multiphase examples will use much more memory compared
to a standard external flow aerodynamics or heat transfer problem. This is
difficult for Ansys to advise as their customer base uses various hardware with
various RAM and CPU core-count configurations. Fluent also does not have a
memory estimator (yet), as given the combination of models, cell types, and
material properties, this is not a trivial problem.

Therefore, as a catch-all rule, 200,000 cells/core for research license holders,
and 100,000 cells/core for student license holders is the recommended metric.
However, one can easily test this by increasing the cells/core count and
observing the scaling by submitting jobs iteratively.

Example
^^^^^^^

Take for instance the cells in the following example mesh are roughly 11 million. 
(Note, the number of cells in Ansys console can be obtained from the command, 
``/report/mesh-size``)

.. code-block:: bash

    >/report/mesh-size
    number of interior nodes = 4893059
    number of interior faces = 6790856
    number of interior cells = 11165614
    number of boundary nodes = 321278
    number of boundary edges = 5145
    number of boundary faces = 112069


For student licenses, this will translate to 11,165,614/100,000 = 111.65 cores
~112 cores, while for research license holders, this will be 11,165,614/200,000 =
55.82 cores ~ 56 cores. For various SLURM partitions, the following should
suffice, (however details about the partitions are also indicated below:)


+--------------+------------+-----------+------------+
| Partitions   | Cores/node | --ntasks  | --nodes    |
+==============+============+===========+============+
|      all     |     96     |     112   |      2     |
+--------------+------------+-----------+------------+

Here, all partitions imply defq, shortq, devq, hmemq, ampereq, ampere-devq, ampere-mq, ampere-m-devq as all of these have 96 cores per node. However, the memory request for these would vary. 


Users need to consider the communication vs computation overhead
associated with their job. What this means is that if a user has not estimated the cells/core metric properly, the runtime will be higher. For instance, taking the above defq partition example, if 2 nodes are used, but only 70 cores are chosen as --ntasks, (while the BATCH partition can max up to 192 cores for 2 nodes, as it has 96 cores/node), the job will now be split across two different
nodes, while remaining heavily under-utilized. This means that Fluent must keep track of and assemble results from two separate nodes, which increases the communication overhead. Simply put, more nodes or cores do not always mean faster results, as there will be a communication overhead that must be accounted for.


Fluent with GPU
---------------


Using GPU acceleration is usually discouraged for research license holders because there are only a few applications within Ansys that benefit from this as per the Ansys customer forum, while for the student licenses, they are not supported at all. Hence they are not recommended. The ideal method recommended for the same, at least for a local installation of Ansys, is briefly explained in this `youtube video <https://www.youtube.com/watch?v=9YH9p2KbRls>`_. However, do check if you possess the license for the same or not.



