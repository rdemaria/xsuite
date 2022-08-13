
=========================
Data management in Xsuite
=========================

Done through the Xobjects package

HybridClasses, xofields, xobjects
=================================

Beam elements and Particles objects are hybrid objects built with the Xobjects
package. They contain, along with standard python attributes and methods,
also an "xobject" that can be optionally stored on GPU and made accessible to
the C code used in the implementation.

The set of attibutes accessible in C and the corresponding types can be found in
```_xofields``` dictionary attached to the class for example:

.. code-block:: python

    xtrack.Multipole._xofields

    # contains:
    # {'order': Int64,
    #  'inv_factorial_order': Float64,
    #  'length': Float64,
    #  'hxl': Float64,
    #  'hyl': Float64,
    #  'radiation_flag': Int64,
    #  'knl': <array ArrNFloat64>,
    #  'ksl': <array ArrNFloat64>,
    #  '_internal_record_id': <struct RecordIdentifier>}

Each instance of the class contains an xobject storint the corresponding data.

For example:

.. code-block:: python

    m = xtrack.Multipole(knl=[1,2,3])

    m._xobject.knl # contains [1,2,3]

All attributes of the xobject are automatically exposed as attributes of the beam element.
For example, ``m._xobject.length`` is the same as ``m.length``.

Arrays are exposed as native Xobjects arrays in the ``_xobject`` attribute, and
as numpy or numpy-like arrays as attributes of the beam element. For example:

.. code-block:: python

    m = xtrack.Multipole(knl=[1,2,3])

    mp._xobject.knl
    # is an xobjects.Array

    mp.knl
    # is a numpy array

It should be noted that the two are different views of the same memory area,
hence any modification can be made indifferently on any of them.

The numpy view (or np-like on GPU contexts) gives the possibility of using
numpy-compatible functions and features on the array (e.g. ``np.sum``, ``np.mean``,
slicing, masking, etc.).

Contexts and buffers
====================

The xobjects are allocated in memory buffers managed by the xobjects package.
Buffers can be allocated on the GPU or on the CPU memory depending on the context.

For example the following code creates two buffer in a GPU memory:

.. code-block:: python

    # We create a GPU context
    context = xobjects.ContextCupy()

    buffer1 = ctx.new_buffer()              # using default initial capacity
    buffer2 = ctx.new_buffer(capacity=1000) # specifying initial capacity

A buffer can contain multiple hybrid objects. For example to allocate two Multipole
objects in buffer1:

.. code-block:: python

    mult1 = xt.Multipole(knl=[1, 2, 3], _buffer=buffer1)
    mult2 = xt.Multipole(knl=[1, 2, 3], _buffer=buffer1)

The capacity of the buffer is automatically increased to fit the allocated objects.

Buffers can also be created implicitly when creating the objects. This is done
by passing the context instead of the buffer. For example:

.. code-block:: python

    context = xobjects.ContextCupy()

    mult1 = xt.Multipole(knl=[1, 2, 3], _context=context)
    mult2 = xt.Multipole(knl=[1, 2, 3], _context=context)

In this case a new buffer is created automatically for each of the objects.

If neither a context nor a buffer is specified, the default context (ContextCpu)
is used.

The buffer and context of an object can be inspected using the ``_buffer`` and
``_context`` attributes:

.. code-block:: python

    mult1._buffer # gives the buffer of the object
    mult2._context # gives the context of the object

Move and copy operations
========================

The ``copy`` method can be used copy the objects across buffers and contexts.
For example:

.. code-block:: python

    # we create two multipoles in the default context
    mult1 = xt.Multipole(knl=[1, 2, 3])
    mult2 = xt.Multipole(knl=[3, 4, 5])

    # We create a GPU context
    context_gpu = xobjects.ContextCupy()

    # We make copy of the first object in a GPU context (a new buffer in the
    # GPU memory is created automatically)
    mult1_gpu = mult1.copy(_context=context_gpu)

    # We make a copy of the second multipole to a specific GPU buffer
    buffer_gpu = context_gpu.new_buffer()
    mult2_gpu = mult2.copy(_buffer=buffer_gpu)

    # It no argument is passed to the copy method, the copy is made in the same
    # context as the original object (a new buffer is created).
    another_copy = mult2_gpu.copy()


The ``move`` method can be used move objects across buffers and contexts.
For example:

.. code-block:: python

    # we create two multipoles in the default context
    mult1 = xt.Multipole(knl=[1, 2, 3])
    mult2 = xt.Multipole(knl=[3, 4, 5])

    # We create a GPU context
    context_gpu = xobjects.ContextCupy()

    # We move the first object in a GPU context (a new buffer in the
    # GPU memory is created automatically)
    mult1.move(_context=context_gpu)

    # We move the second object to a specific GPU buffer
    buffer_gpu = context_gpu.new_buffer()
    mult2.move(_buffer=buffer_gpu)








