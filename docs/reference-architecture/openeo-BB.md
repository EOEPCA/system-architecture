# openEO Building Block

The openEO BB provides an implementation of a _Processing BB_ as introduced in the [Processing Concepts][processing-concepts] section. It provides an implementation of the openEO API Specification [[RD19]][rd19], to support the remote execution of openEO Process Graphs [[RD20]][rd20].

There is an opportunity to engineer the openEO building block in such a way that it also facilitates consumption through users of the Pangeo stack. Several measures are identified that may aid this interoperation:

*	Processing Runner that is based on Dask
*	Processing Client providing an implementation of the Xarray backend through which the openEO service be consumed as Xarray datasets
*	Use of Zarr for data representation

Processing Client tooling already exists within the openEO stack, with support for Python, JavaScript and R.
