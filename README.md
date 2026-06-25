# operon-skills


##	C4 Cluster

You will need to set up an actual ssh connection in your `~/.ssh/config` file with something like ...

```bash

Host c4-log1
	User YOUR_USERNAME
	HostName c4-log1.ucsf.edu

Host c4-log1-dev4
	HostName c4-dev4
	User YOUR_USERNAME
	ProxyJump c4-log1

```

Then you'll need to add this Compute -> SSH Host





##	Skills

###	Connections

####	C4 Cluster

This will include some specific standards to know

```bash

- **scratch_root must be shared storage.** `/scratch/$USER` is node-local (created per-job as `$TMPDIR`), not visible across login ↔ compute nodes. Use `/francislab/data1/operon` instead. The probe auto-detects `/scratch/gwendt` — if re-probed, manually re-edit `scratch_root` back to `/francislab/data1/operon`.
- **Do not specify `--partition` or `--account`** in SBATCH directives — the default works. Specifying `--account=common` with partition `common` caused `PartitionConfig` failures.

- **Do not use conda.** Use `module load`, `pip install --user`, `venv`, or apptainer containers instead.
- Always inform user before installing anything — installs are permanent under their account (gwendt).

- **Use Python venvs** (not conda) to keep installs clean.
- cluster nodes do not have internet access so any package installs must be done via the connected server

**Compute nodes have NO internet access. Never run `BiocManager::install` / `install.packages` / `pip install` / `remotes::install_github` inside `c.submit_job`** — even with the `francislab` default partition, the SLURM job runs on c4-n17 (or another compute node) and the install will fail with proxy/network errors. Per-user policy.

Installs must run via `c.call_command()` on the login node. Long installs (10-20 min for the full Bioc methylation stack) exceed the 600s `call_command` cap, so:
- Launch the install as a backgrounded `nohup ... &` from `call_command` (returns within seconds after spawning).
- Write outputs to a marker file (`install_*.done`).
- Poll progress in subsequent `call_command` calls until the marker file appears.
c4-n1 and c4-n2 are problematic shared nodes — must be excluded from every submit_job. Without an explicit constraint, SLURM may schedule jobs there. The c4-n17 affinity from prior session was apparently a user-controlled hint, not a hard requirement encoded in our submits.

**Action item**: every SBATCH header from now on must contain:

#SBATCH --exclude=c4-n1,c4-n2


Include heartbeats in all scripts



Log enough to know the progress and enough results to understand what has or has not happened
No extended silent periods without explicit notification
Add a heartbeat to ensure progress
Add a plan and progress file that an outside observer could use to monitor the progress of this run.


```



#### Datasets

For each, include ...

* Raw data location and format
* Metadata covariates
* data dictionary if available
* pipeline outputs


##### 1000 genomes

##### AGS

##### GTEx

##### IPS

##### Target

##### TCGA


