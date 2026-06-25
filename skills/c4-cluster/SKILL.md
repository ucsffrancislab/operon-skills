
---
name: c4-cluster
description: C4 Cluster description and general usage rules for the login, development and cluster nodes. C4 is a shared high-performance compute (HPC) cluster underlying supporting the Helen Diller Family Comprehensive Cancer Center (HDFCCC). Funded by the HDFCCC and administered by the Computational Biology and Informatics (CBI) Shared Resource, it is available to all UCSF Cancer Center members and members of their labs. Its purpose is to fulfill various biomedical and health science computing needs. Researchers can participate using the common resources of this system or buy compute nodes or storage to add to the system and for which special priority is given.
---

Depending on how the user has setup the connection, this network currently contains:

* 2 login nodes: c4-log1 and c4-log2
  * They have minimal resources.
  * They do not have `module` access.
  * They should really not be used for anything other than submitting jobs.
* 2 development nodes: c4-dev3 and c4-dev4
  * They are shared and that must be respected.
  * They have internet access and should be used when installing packages.
* 2 data transfer nodes: c4-dt1 and c4-dt2
  * They have minimal resources.
  * They do not have `module` access.
  * They have internet access.
  * They are only good for large data transfers in or out.
  * They really should just be ignored.
* Many cluster compute nodes
  * They have `module` access.
  * They DO NOT have internet access.


My preference is to set the connection to always land on c4-dev4.

C4 is a slurm cluster.

It is advised that any script submitted as a job contain the following header with directives:

```bash
# Do not let the current environment leak into the job
#SBATCH --export=NONE
# c4-n1 and c4-n2 have been problematic but still exist. Avoid using ...
#SBATCH --exclude=c4-n1,c4-n2
```




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

Include heartbeats in all scripts

Log enough to know the progress and enough results to understand what has or has not happened
No extended silent periods without explicit notification
Add a heartbeat to ensure progress
Add a plan and progress file that an outside observer could use to monitor the progress of this run.

```




