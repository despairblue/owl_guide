Modules & Basics
=================================================

This chapter helps you to warm up with some basic concepts and module structures in Owl. We will delve deeper into some of the topics in future.

For both n-dimensional array and matrix, Owl supports: both dense and sparse data structures; both single and double precisions; both real and complex numbers. The dense data structure is built directly atop of OCaml's native Bigarray library, so if another library also works on Bigarray, exchanging date between both will be trivial.

In the following examples, I assume that you have already loaded Owl library with ``#require "owl"``, and opened Owl module with ``open Owl`` in ``utop``. If you don't have Owl installed locally yet, you can still try the examples by pulling in a ready-made docker image of the latest `Owl` with the following commands.

.. code-block:: bash

  docker pull ryanrhymes/owl
  docker run -t -i ryanrhymes/owl


Alternatively, it is recommended to have a full installation for the best performance, please refer to the :doc:`Installation Guide <./install>`. OK, let's start.



Module Structure
-------------------------------------------------


Dense & Sparse
-------------------------------------------------

In Owl, ``Dense`` module contains the modules of dense data structures. For example, ``Dense.Matrix`` supports the operations of dense matrices. Similarly, ``Sparse`` module contains the modules of sparse data structures.

.. code-block:: ocaml

  Dense.Ndarray   (* dense ndarray *)
  Dense.Matrix    (* dense matrix *)

  Sparse.Ndarray  (* sparse ndarray *)
  Sparse.Matrix   (* sparse ndarray *)


All these four modules consists of five submodules to handle different types of numbers.

* ``S`` module supports single precision float numbers `float32`;
* ``D`` module supports double precision float numbers `float64`;
* ``C`` module supports single precision complex numbers `complex32`;
* ``Z`` module supports double precision complex numbers `complex64`;
* ``Generic`` module supports all aforementioned number types via GADT.

With ``Dense.Ndarray``, you can create a dense n-dimensional array of no more than 16 dimensions. This constraint originates from the underlying `Bigarray.Genarray` module. In practice, this constraint makes sense since the space requirement will explode as the dimension increases. If you need anything higher than 16 dimensions, you need to use ``Sparse.Ndarray`` to create a sparse data structure.


Number & Precision
-------------------------------------------------

After deciding the suitable data structure (either dense or sparse), you can create a ndarray/matrix using creation function in the modules, using e.g., ``empty``, ``create``, ``zeros``, ``ones`` ... The type of numbers (real or complex) and its precision (single or double) needs to be passed to the creations functions as the parameters.

.. code-block:: ocaml

  Dense.Ndarray.Generic.zeros Float64 [|5;5|];;

With ``zeros`` function, all the elements in the created data structure will be initialised to zeros.

Technically, ``S``, ``D``, ``C``, and ``Z`` are the wrappers of ``Generic`` module with explicit type information provided. Therefore you can save the type constructor which was passed into the ``Generic`` module if you use these submodules directly.

.. code-block:: ocaml

  Dense.Ndarray.S.zeros [|5;5|];;    (* single precision real ndarray *)
  Dense.Ndarray.D.zeros [|5;5|];;    (* double precision real ndarray *)
  Dense.Ndarray.C.zeros [|5;5|];;    (* single precision complex ndarray *)
  Dense.Ndarray.Z.zeros [|5;5|];;    (* double precision complex ndarray *)


The following examples are for dense matrices.

.. code-block:: ocaml

  Dense.Matrix.S.zeros 5 5;;     (* single precision real matrix *)
  Dense.Matrix.D.zeros 5 5;;     (* double precision real matrix *)
  Dense.Matrix.C.zeros 5 5;;     (* single precision complex matrix *)
  Dense.Matrix.Z.zeros 5 5;;     (* double precision complex matrix *)


The following examples are for sparse ndarrays.

.. code-block:: ocaml

  Sparse.Ndarray.S.zeros [|5;5|];;    (* single precision real ndarray *)
  Sparse.Ndarray.D.zeros [|5;5|];;    (* double precision real ndarray *)
  Sparse.Ndarray.C.zeros [|5;5|];;    (* single precision complex ndarray *)
  Sparse.Ndarray.Z.zeros [|5;5|];;    (* double precision complex ndarray *)


The following examples are for sparse matrices.

.. code-block:: ocaml

  Sparse.Matrix.S.zeros 5 5;;     (* single precision real matrix *)
  Sparse.Matrix.D.zeros 5 5;;     (* double precision real matrix *)
  Sparse.Matrix.C.zeros 5 5;;     (* single precision complex matrix *)
  Sparse.Matrix.Z.zeros 5 5;;     (* double precision complex matrix *)


In short, ``Generic`` module can do everything that submodules can, but for some functions (e.g. creation functions) you need to explicitly pass in the type information.



Polymorphic Functions
-------------------------------------------------

Polymorphism is achieved by pattern matching and GADT in ``Generic`` module. This means many functions in ``Generic`` module can handle aforementioned four different number types.

In the following, I use the `sum` function in `Dense.Matrix.Generic` module as an example. `sum` function returns the summation of all the elements in a matrix.

.. code-block:: ocaml

  let x = Dense.Matrix.S.eye 5 in
    Dense.Matrix.Generic.sum x;;

  let x = Dense.Matrix.D.eye 5 in
    Dense.Matrix.Generic.sum x;;

  let x = Dense.Matrix.C.eye 5 in
    Dense.Matrix.Generic.sum x;;

  let x = Dense.Matrix.Z.eye 5 in
    Dense.Matrix.Generic.sum x;;


As we can see, no matter what kind of numbers are held in an identity matrix, we always pass it to ``Dense.Matrix.Generic.sum`` function. Similarly, we can do the same thing for other modules (``Dense.Ndarray``, ``Sparse.Matrix``, and etc.) and other functions (``add``, ``mul``, ``neg``, and etc.).

Meanwhile, each submodule also contains the same set of functions, e.g, as below,

.. code-block:: ocaml

  Dense.Matrix.S.(eye 5 |> sum);;



Shortcuts to Double Precision Modules
-------------------------------------------------

In reality, we often work with double precision numbers, therefore Owl provides some shortcuts to the data structures of double precision float numbers:

* ``Arr`` is equivalent to double precision real ``Dense.Ndarray.D``;
* ``Mat`` is equivalent to double precision real ``Dense.Matrix.D``;

With these shortcut modules, you are no longer required to pass in type information. Here are some examples.

.. code-block:: ocaml

  Arr.zeros [|5|];;        (* same as Dense.Ndarray.D.zeros [|5|] *)
  Mat.zeros 5 5;;          (* same as Dense.Matrix.D.zeros 5 5 *)
  ...


More examples besides creation functions are as follows.

.. code-block:: ocaml

  Mat.load "data.mat";;    (* same as Dense.Matrix.D.load "data.mat" *)
  Mat.of_array 5 5 x;;     (* same as Dense.Matrix.D.of_array 5 5 x *)
  Mat.linspace 0. 9. 10;;  (* same as Dense.Matrix.D.linspace 0. 9. 10 *)
  ...


If you actually work more often with other number types like Complex, you can certainly make your own alias to corresponding ``S``, ``D``, ``C``, and ``Z`` module if you like.



Casting into Another Type
-------------------------------------------------

As I mentioned before, there are four basic number types. You can therefore cast one value from one type to another one by using the ``cast_*`` functions in ``Generic`` module.

* ``Generic.cast_s2d``: cast from ``float32`` to ``float64``;
* ``Generic.cast_d2s``: cast from ``float64`` to ``float32``;
* ``Generic.cast_c2z``: cast from ``complex32`` to ``complex64``;
* ``Generic.cast_z2c``: cast from ``complex64`` to ``complex32``;
* ``Generic.cast_s2c``: cast from ``float32`` to ``complex32``;
* ``Generic.cast_d2z``: cast from ``float64`` to ``complex64``;
* ``Generic.cast_s2z``: cast from ``float32`` to ``complex64``;
* ``Generic.cast_d2c``: cast from ``float64`` to ``complex32``;

In fact, all these function rely on the following ``cast`` function.

.. code-block:: ocaml

  val cast : ('a, 'b) kind -> ('c, 'd) t -> ('a, 'b) t


The first parameter specifies the cast type. If the source type and the cast type are the same, ``cast`` function simply makes a copy of the passed in value.

.. code-block:: ocaml

  let x = Arr.uniform [|8;8|];;                     (* created in float64 *)
  let y = Dense.Ndarray.Generic.cast Complex32 x;;  (* cast to complex32 *)



More in Documents
-------------------------------------------------

To know more about the functions provided in each module, please read the corresponding interface file of `Generic` module. The ``Generic`` module contains the documentation.

* `Dense.Ndarray.Generic <https://github.com/ryanrhymes/owl/blob/master/src/owl/dense/owl_dense_ndarray_generic.mli>`_
* `Dense.Matrix.Generic <https://github.com/ryanrhymes/owl/blob/master/src/owl/dense/owl_dense_matrix_generic.mli>`_
* `Sparse.Ndarray.Generic <https://github.com/ryanrhymes/owl/blob/master/src/owl/sparse/owl_sparse_ndarray_generic.mli>`_
* `Sparse.Matrix.Generic <https://github.com/ryanrhymes/owl/blob/master/src/owl/sparse/owl_sparse_matrix_generic.mli>`_
