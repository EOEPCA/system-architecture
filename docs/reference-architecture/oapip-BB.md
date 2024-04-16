# OAPIP Building Block

The OAPIP building-block provides an implementation of the OGC API Processes standard – Parts 1 & 2(draft) [RD05/RD06] in conjunction with the OGC Best Practice for Application Packages [[RD18]][rd18].

The building-block should support extensible Processing Runner implementations – noting that the current EOEPCA ADES component (ZOO-Project-DRU) provides a CWL runner (Calrissian) that executes on Kubernetes. Work has also been performed to integrate an alternative CWL Runner (TOIL) that executes on Slurm HPC.

The OAPIP building-block should support the integration of steps within the Application Package CWL [[RD18]][rd18] that exploit other processing backend technologies, including:

*	Steps that execute openEO Process Graphs [[RD20]][rd20]
*	Steps that execute Dask Task Graphs

The programmatic Processing Client should facilitate interaction with the OGC API Processes Processing Engine, including dynamically establishing (i.e. in code) the OGC Application Package CWL [[RD18]][rd18] workflow steps for execution.
