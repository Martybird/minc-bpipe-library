Library of bpipe functions for processing minc files
====================================================

minc-bpipe-library provides a set of chainable minc file processing functions to (pre)process data.

It requires http://www.bic.mni.mcgill.ca/ServicesSoftware/ServicesSoftwareMincToolKit, https://stnava.github.io/ANTs/, and https://github.com/ssadedin/bpipe/

To control which stages are run, edit ``pipeline.bpipe`` and add stage names using "+" to the "run" stage.

The default in ``pipeline.bpipe`` is what has experimentally determined to be a best-practice run.

Stages are listed in the ``minc-library.bpipe``, correction stages such as denoising, n3 and n4, and normalize
should be run before any other stages. Typically after this ``linear_antsRegistration`` would be run, which will
allow other processing such as VBM, cutneck, deface and beast to be done in MNI space.

For convenience, "segments" have been defined for some processing jobs (beast, cutneck, VBM) which are multi-step.
These may conflict with each other if you try to combine them, so instead you must specifiy their individual stages.

Stage options have been chosen based on best pratices from publications where applicable but can be changed.

Once you have defined your stages, you can run your pipeline on the SGE cluster with:
```
> git clone https://github.com/CobraLab/minc-bpipe-library.git
#Edit minc-bpipe-library/pipeline.bpipe as you see fit
> module load bpipe #needed to run bpipe
> module load minc-toolkit/1.9.10 #needed for most stages
> cd /path/to/store/outputs
#Choose n to be the smaller of (number of input files, 240)
> bpipe run -n<number> /path/to/pipeline.bpipe /path/to/inputs/*mnc
```

Output filenames from bpipe will be of the form ``<inputname>.stage1.stage2.stage3.ext`` where the last stage
name will be the last run. ``commandlog.txt`` will be generated by bpipe describing all the commands run for
the pipeline. This is a very good file to keep around as as a note of what happened to your data.

Some stages produce both ``mnc`` files and ``xfm`` files, see the ``$output`` variables for what is generated.

#Scinet Operation
New (experimental) operation on SciNet is now implemented. Inputs are split into 8 file chunks and submitted
as local bpipe jobs on scinet nodes

Steps

1. ``git clone https://github.com/CobraLab/minc-bpipe-library.git``
2. ``rm minc-bpipe-library/bpipe.config``
3. ``sed -i 's#/opt/quarantine#/project/m/mchakrav/quarantine#g' minc-bpipe-library/minc-library.bpipe``
4. ``module load scinet``
5. Use ``bpipe-batch.sh /path/to/pipeline.bpipe <list of files> > joblist`` to generate a joblist
6. Use ``qbatch joblist 1 12:00:00`` to submit jobs to scinet queing system
