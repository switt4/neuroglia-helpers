# neuroglia-helpers

Helper & wrapper scripts for using Singularity images locally and remotely on graham (compute canada) - or any other SLURM system

Features:
* wrapper scripts for submitting single and batch jobs using SLURM arrays
* variants of scripts that use (neuroglia*) and do not use (regular*) singularity
* scripts for interactive job submission including singularity shell
* BIDS-App support with parallelization over subjects with SLURM arrays
* Deployment via docker2singularity or singularity-hub

## Install:

To set-up neuroglia-helpers on graham, run the following:
```
git clone http://github.com/khanlab/neuroglia-helpers ~/neuroglia-helpers
echo "export PATH=~/neuroglia-helpers:\$PATH" >> ~/.bashrc
echo "export SINGULARITY_DIR=/project/6007967/akhanf/singularity" >> ~/.bashrc
echo "export SINGULARITY_IMG=\${SINGULARITY_DIR}/khanlab_neuroglia-vasst-dev_0.0.2f.img" >> ~/.bashrc
echo "export SINGULARITY_OPTS=\"-e -B /cvmfs:/cvmfs -B /project:/project -B /scratch:/scratch\"" >> ~/.bashrc
```

To set-up singularity 2.5 on graham, run the following:
```
echo "module load singularity/2.5" >> ~/.bashrc
```

### Setting up your workspace:

Compute Canada does not allow any GUI (X) applications to be run, even in interactive mode. The best way to run these applications is from your local workstation, using sshfs to mount your folder on graham, (and optionally passwordless ssh to allow for easier access). Use the setup_* scripts to configure your local system. Once your sshfs mount is created, you can use that for visualizing your data, running stats, etc., latency is reasonable as long as data is not too big (i.e. GB's)

## BIDS Apps:

The bidsBatch wrapper uses SLURM to parallelize over *subjects*, and by default will run a job for every subject in your participants.tsv file. 
bidsBatch uses SLURM Arrays, so it groups all the different jobs under a single job ID, with each array job indicated as <jobid>_<index>.

Running bidsBatch lists the usage and displays bids-apps that are deployed on the system (contents of bids-apps.tsv).  For default app options, please also see the bids-apps.tsv file.  Note: when you run bidsBatch to process a dataset, a ```jobs/``` folder is created in the output directory you specify.


### Example: Running fmriprep on your bids dataset
```
bidsBatch fmriprep_1.0.4 ~/my-bids-dataset ~/my-bids-dataset/derivatives/fmriprep-v1.0.4 participant 
```
To get usage/options for a particular bids app, leave out the bids-app required arguments, e.g.:

```
bidsBatch fmriprep_1.0.4 
```



## Wrappers:

Wrappers for singularity that make use of the following environment variables in your environment (set in .bashrc):
```
SINGULARITY_DIR=<path to folder containing singularity images>
SINGULARITY_IMG=<path to default (neuroglia) singularity container>
SINGULARITY_OPTS=<options for singularity, e.g. path binding>
```


* neuroglia

Simple wrapper for singularity exec (does not submit a job)

* neurogliaShell

Simple wrapper for singularity shell (does not submit a job)

* neurogliaInteractive

Wrapper for submitting an interactive cluster job, using a singularity container
Note: interactive job default is 1hour, 8cores, 32GB memory

* neurogliaBatch

Wrapper for submitting batch cluster jobs, using a singularity container

```
==========================================================================
Interface for running pipeline scripts on the cluster with singularity

  Loops through an input subject_list_txt (txt with subj id on each line)
  Can be used to run scripts that generally take command-line parameters as:

  <script_name> <before args (optional)> <subjid (required)> <after args (optional)>

--------------------------------------------------------------------------
Usage: neurogliaBatch  <script_name>  <subject_list_txt>  <optional flags> 
--------------------------------------------------------------------------

optional flags:

 -g : group/reduce job, pass the subject list instead of looping over subjects
 -s <subjid> : single-subject mode, run on a single subject (must be in subjlist) instead
 -t : test-mode, don/'t actually submit any jobs

 Required resources:
 -j <job-template> :  sets requested resources
	Regular (default):	8core/32gb/24h
	LongSkinny:		1core/4gb/72h
	ShortFat:		32core/128gb/3h

 Passing arguments to pipeline script
  -b /args/ : cmd-line arguments that go *before* the subject id
  -a /args/ : cmd-line arguments that go *after* the subject id

 SLURM Job dependencies for pipelining (man sbatch for more details):
  -d aftercorr:jobid[:jobid]	: each subj depends on completed subj in submitted job ids
  -d afterok:jobid[:jobid]	: each subj depends on all completed subj in submitted job id
```




