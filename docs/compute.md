---
title: Compute Resources
layout: default
nav_order: 4
description: CPU, memory, GPU, networking and attached storage, and how they differ on a laptop, a cloud instance, and a university HPC cluster.
---

# Compute Basics
{: .fs-9 }

Every machine you will ever run code on is the same five parts in different
amounts. A laptop, a rented cloud instance, and a node in the university
cluster all give you processors, memory, sometimes a GPU, a network connection,
and a disk. What changes is how much of each you get, how long you wait for it,
who else is using it, and who pays.
{: .fs-6 .fw-300 }

<svg viewBox="0 0 640 320" width="100%" style="max-width:640px;height:auto;display:block;margin:1.5rem auto;" role="img" aria-labelledby="node-title">
  <title id="node-title">The parts of one machine: CPU, memory, GPU with its own VRAM, attached disk and network, and the links between them</title>
  <g font-family="system-ui,sans-serif">
    <text x="20" y="34" fill="#232D4B" font-size="13" font-weight="700">one machine: laptop, cloud instance, or cluster node</text>
    <rect x="20" y="44" width="600" height="226" rx="8" fill="none" stroke="#232D4B" stroke-width="2"/>
    <rect x="44" y="70" width="146" height="86" rx="6" fill="#232D4B"/>
    <text x="117" y="92" text-anchor="middle" fill="#ffffff" font-size="14" font-weight="700">CPU</text>
    <g fill="#5b6478" stroke="#c9d1e3" stroke-width="1">
      <rect x="81" y="102" width="14" height="14" rx="2"/><rect x="99" y="102" width="14" height="14" rx="2"/>
      <rect x="117" y="102" width="14" height="14" rx="2"/><rect x="135" y="102" width="14" height="14" rx="2"/>
      <rect x="81" y="122" width="14" height="14" rx="2"/><rect x="99" y="122" width="14" height="14" rx="2"/>
      <rect x="117" y="122" width="14" height="14" rx="2"/><rect x="135" y="122" width="14" height="14" rx="2"/>
    </g>
    <text x="117" y="150" text-anchor="middle" fill="#c9d1e3" font-size="11">8 cores</text>
    <rect x="232" y="70" width="146" height="86" rx="6" fill="#dbe3f0" stroke="#232D4B" stroke-width="2"/>
    <text x="305" y="95" text-anchor="middle" fill="#232D4B" font-size="14" font-weight="700">Memory</text>
    <text x="305" y="116" text-anchor="middle" fill="#232D4B" font-size="12">32 GB</text>
    <text x="305" y="140" text-anchor="middle" fill="#5b6478" font-size="10.5">where your DataFrame lives</text>
    <line x1="190" y1="113" x2="232" y2="113" stroke="#E57200" stroke-width="3"/>
    <text x="211" y="105" text-anchor="middle" fill="#E57200" font-size="9.5" font-weight="600">fast</text>
    <rect x="420" y="70" width="176" height="120" rx="6" fill="#232D4B"/>
    <text x="508" y="92" text-anchor="middle" fill="#ffffff" font-size="14" font-weight="700">GPU</text>
    <rect x="436" y="104" width="144" height="38" rx="4" fill="#3a4670" stroke="#E57200" stroke-width="1.5"/>
    <text x="508" y="128" text-anchor="middle" fill="#ffffff" font-size="11.5">VRAM 24 GB</text>
    <text x="508" y="164" text-anchor="middle" fill="#c9d1e3" font-size="10.5">thousands of small cores</text>
    <line x1="378" y1="113" x2="420" y2="113" stroke="#E57200" stroke-width="3"/>
    <text x="399" y="105" text-anchor="middle" fill="#E57200" font-size="9.5" font-weight="600">PCIe</text>
    <rect x="44" y="196" width="146" height="56" rx="6" fill="#ffffff" stroke="#232D4B" stroke-width="2"/>
    <text x="117" y="218" text-anchor="middle" fill="#232D4B" font-size="12" font-weight="700">Attached disk</text>
    <text x="117" y="236" text-anchor="middle" fill="#5b6478" font-size="10.5">NVMe, 1 TB</text>
    <rect x="232" y="196" width="146" height="56" rx="6" fill="#ffffff" stroke="#232D4B" stroke-width="2"/>
    <text x="305" y="218" text-anchor="middle" fill="#232D4B" font-size="12" font-weight="700">Network</text>
    <text x="305" y="236" text-anchor="middle" fill="#5b6478" font-size="10.5">10 Gbps</text>
    <line x1="117" y1="156" x2="117" y2="196" stroke="#232D4B" stroke-width="2"/>
    <line x1="305" y1="156" x2="305" y2="196" stroke="#232D4B" stroke-width="2"/>
    <text x="320" y="290" text-anchor="middle" fill="#6b7280" font-size="12">Each link runs at a different speed. Memory is quickest, then PCIe to the GPU,</text>
    <text x="320" y="308" text-anchor="middle" fill="#6b7280" font-size="12">then the local disk, then the network. Your slowest link sets your runtime.</text>
  </g>
</svg>

---

## CPU

The processor runs your Python. Cores are the part worth understanding: a core
executes one stream of instructions at a time, so eight cores can do eight
things at once, and one core does one thing very quickly.

The catch that surprises people is that plain Python code uses exactly one core.
A `for` loop over a million rows will peg one core at 100% and leave the other
seven idle no matter how many you paid for. What does use them is library code
written in C underneath: NumPy matrix operations, many scikit-learn estimators
with `n_jobs=-1`, Polars, DuckDB, and anything you explicitly parallelize with
`joblib` or `multiprocessing`. Before you rent a 64-core machine, check whether
the thing you're waiting on can use more than one.

Clock speed still matters for the single-threaded parts, which is why a fast
laptop sometimes beats a big cloud instance on a task that never parallelizes.

## Memory

RAM is where your data sits while you work on it, and running out of it is the
single most common way a data science job dies. The failure is abrupt: a
`MemoryError`, or the operating system killing your process outright, or on a
cluster the scheduler ending your job with an out-of-memory message.

The number to internalize is that a DataFrame is much larger than the file it
came from. Budget several times the size of a CSV once it's parsed into memory,
commonly five to ten times, because text becomes typed columns and Python
objects carry overhead. A 4 GB CSV on a 16 GB laptop is a genuine risk, and the
`groupby` that copies it is what tips it over.

The fixes, roughly in order of effort: load only the columns you need, set
smaller dtypes, store the file as Parquet instead of CSV, process it in chunks,
or switch to a tool built to work out of core such as Polars or DuckDB. Renting
more memory is also a completely legitimate answer, and often the cheapest one
measured in your time.

## GPU

A GPU is a different shape of processor. Instead of eight fast general-purpose
cores it has thousands of small ones that all do the same operation to different
data at the same time. That suits matrix multiplication enormously, which is why
deep learning runs on GPUs and why training that takes a week on a CPU can take
an afternoon on one.

Two things trip students up. The first is that a GPU has its own memory, and it
is smaller than your system RAM. A model plus its activations plus a batch of
data all have to fit in that VRAM, which is where "CUDA out of memory" comes
from, and why the first thing people try is a smaller batch size. The second is
that data has to be copied from system memory across the PCIe link before the
GPU can touch it. If your GPU sits at 4% utilization while your job crawls, you
are almost certainly feeding it too slowly rather than lacking GPU power.

Most classical data science gets nothing from a GPU. Pandas, scikit-learn, XGBoost
on modest data, and every SQL query you write are CPU work. Rent a GPU when
you're training neural networks, running a model for inference at volume, or
using a library built for it such as PyTorch, JAX, or cuDF.

## Networking

Two different numbers hide under "network." Bandwidth is how much data you can
move per second, and latency is how long a single round trip takes. Downloading
a 200 GB dataset is a bandwidth problem. A training loop that fetches one small
file at a time from S3 is a latency problem, and adding bandwidth will not fix
it.

The practical rule is to move the compute to the data rather than the data to
the compute. Pulling 500 GB from a cloud bucket down to your laptop over campus
wifi is slow and, if it leaves the cloud provider, billed at around $0.09 per
GB. Running the same job on an instance in the same region as the bucket is
faster and costs nothing in egress. See
[Storage]({{ site.baseurl }}/docs/storage.html) for why that bucket is usually
where the data lives in the first place.

On a cluster, the network between nodes is a specialized fabric such as
InfiniBand, which is what makes a job that spans forty machines possible. You
only care if you're running genuinely distributed work.

## Attached storage

The disk on the machine is fast and temporary in ways that matter. On a cloud
instance you generally have a network-attached volume that persists when you
stop the instance, and sometimes a local NVMe scratch disk that is much faster
and is erased the moment the instance goes away. Knowing which one you're
writing to is worth thirty seconds of checking.

On an HPC cluster you'll meet at least two filesystems with different rules.
Home directories are small, backed up, and meant for code and configuration.
Scratch space is large and fast, is not backed up, and is usually purged
automatically after some number of days. Every semester someone loses results
they left in scratch. Write your outputs somewhere durable when the job
finishes.

---

## Where you run it

<svg viewBox="0 0 640 300" width="100%" style="max-width:640px;height:auto;display:block;margin:1.5rem auto;" role="img" aria-labelledby="scale-title">
  <title id="scale-title">Typical resources on a laptop, a cloud instance, and an HPC cluster node</title>
  <g font-family="system-ui,sans-serif">
    <rect x="20" y="50" width="180" height="32" rx="6" fill="#232D4B"/>
    <text x="110" y="71" text-anchor="middle" fill="#ffffff" font-size="13" font-weight="700">Your laptop</text>
    <rect x="20" y="82" width="180" height="170" fill="#dbe3f0" stroke="#232D4B" stroke-width="1.5"/>
    <g font-size="11" fill="#232D4B">
      <text x="34" y="104">8 to 12 cores</text>
      <text x="34" y="126">16 to 64 GB RAM</text>
      <text x="34" y="148">small GPU, maybe</text>
      <text x="34" y="170">1 TB built-in SSD</text>
      <text x="34" y="192">whatever wifi gives</text>
      <text x="34" y="214" font-weight="700">costs nothing extra</text>
      <text x="34" y="236" font-weight="700">available right now</text>
    </g>
    <rect x="230" y="50" width="180" height="32" rx="6" fill="#232D4B"/>
    <text x="320" y="71" text-anchor="middle" fill="#ffffff" font-size="13" font-weight="700">Cloud instance</text>
    <rect x="230" y="82" width="180" height="170" fill="#dbe3f0" stroke="#232D4B" stroke-width="1.5"/>
    <g font-size="11" fill="#232D4B">
      <text x="244" y="104">2 to 192 cores</text>
      <text x="244" y="126">up to a few TB RAM</text>
      <text x="244" y="148">rent any GPU made</text>
      <text x="244" y="170">any size volume</text>
      <text x="244" y="192">10 to 100 Gbps</text>
      <text x="244" y="214" font-weight="700">billed by the second</text>
      <text x="244" y="236" font-weight="700">ready in a minute</text>
    </g>
    <rect x="440" y="50" width="180" height="32" rx="6" fill="#232D4B"/>
    <text x="530" y="71" text-anchor="middle" fill="#ffffff" font-size="13" font-weight="700">HPC cluster</text>
    <rect x="440" y="82" width="180" height="170" fill="#dbe3f0" stroke="#232D4B" stroke-width="1.5"/>
    <g font-size="11" fill="#232D4B">
      <text x="454" y="104">hundreds of nodes</text>
      <text x="454" y="126">big RAM per node</text>
      <text x="454" y="148">shared GPU pool</text>
      <text x="454" y="170">home plus scratch</text>
      <text x="454" y="192">InfiniBand between</text>
      <text x="454" y="214" font-weight="700">no bill to you</text>
      <text x="454" y="236" font-weight="700">you wait in a queue</text>
    </g>
    <text x="320" y="280" text-anchor="middle" fill="#6b7280" font-size="12">The same five resources everywhere. What changes is scale, waiting, and who pays.</text>
  </g>
</svg>

### Your laptop

Start here. It's already yours, nothing has to be requested, and for most
coursework it's enough. Working locally also keeps the feedback loop tight,
which matters more than raw speed while you're still figuring out what the code
should do.

It runs out at predictable places. Memory is the usual wall, and you cannot add
any. Sustained work makes laptops hot, and a hot laptop slows itself down on
purpose, so a job that benchmarks well for two minutes can crawl after twenty.
Closing the lid or losing wifi ends things. If you have an Apple Silicon
machine, the CPU and GPU share one pool of memory, which is unusual and
occasionally very handy, but PyTorch support through the `mps` backend is not as
complete as CUDA on Linux.

Use it for: developing, debugging, plotting, anything under a few GB.

### A cloud instance

You rent a machine by the second, pick exactly the shape you want, and get root
on it. Need 500 GB of RAM for one afternoon? That exists and it's a form to fill
in. This is also the only realistic way most people get access to a current
datacenter GPU on demand.

The costs are real and they're mostly about attention. A stopped instance still
bills you for its attached volume, and a running instance you forgot about bills
you for everything. GPU instances run from roughly a dollar an hour to several
tens of dollars an hour, which turns a weekend of forgetting into an unpleasant
conversation. Spot instances cost far less with the catch that the provider can
take the machine back with a couple of minutes of warning, which is fine for
work that checkpoints and fatal for work that doesn't. And you are now a system
administrator: the machine is yours to patch, secure, and clean up.

Use it for: work that needs a specific machine shape, GPU jobs, anything that
should run near data already in the cloud.

### The university HPC cluster

A cluster is many nodes plus a scheduler that decides who runs what and when.
You do not log into a node and start your job. You log into a login node, which
is a shared front door where running heavy work will annoy hundreds of people,
and you hand a script to the scheduler describing what you need. Slurm is the
scheduler you'll almost certainly meet, through `sbatch` to submit, `squeue` to
see where you are in line, `scancel` to give up, and `sacct` afterwards to find
out what your job actually used.

<svg viewBox="0 0 640 250" width="100%" style="max-width:640px;height:auto;display:block;margin:1.5rem auto;" role="img" aria-labelledby="hpc-title">
  <title id="hpc-title">Work flows from your laptop through a login node to the scheduler queue, which dispatches jobs to compute nodes</title>
  <g font-family="system-ui,sans-serif">
    <rect x="20" y="100" width="100" height="50" rx="6" fill="#ffffff" stroke="#232D4B" stroke-width="2"/>
    <text x="70" y="130" text-anchor="middle" fill="#232D4B" font-size="12">your laptop</text>
    <line x1="120" y1="125" x2="162" y2="125" stroke="#232D4B" stroke-width="2"/>
    <polygon points="162,120 170,125 162,130" fill="#232D4B"/>
    <text x="145" y="115" text-anchor="middle" fill="#5b6478" font-size="10">ssh</text>
    <rect x="170" y="95" width="110" height="60" rx="6" fill="#232D4B"/>
    <text x="225" y="120" text-anchor="middle" fill="#ffffff" font-size="12" font-weight="600">login node</text>
    <text x="225" y="138" text-anchor="middle" fill="#c9d1e3" font-size="9.5">shared, no big jobs</text>
    <line x1="280" y1="125" x2="322" y2="125" stroke="#232D4B" stroke-width="2"/>
    <polygon points="322,120 330,125 322,130" fill="#232D4B"/>
    <text x="305" y="115" text-anchor="middle" fill="#5b6478" font-size="10">sbatch</text>
    <rect x="330" y="70" width="120" height="110" rx="6" fill="#dbe3f0" stroke="#232D4B" stroke-width="2"/>
    <text x="390" y="88" text-anchor="middle" fill="#232D4B" font-size="12" font-weight="700">queue</text>
    <g font-family="ui-monospace,monospace" font-size="10">
      <rect x="344" y="96" width="92" height="20" rx="3" fill="#ffffff" stroke="#232D4B" stroke-width="1"/>
      <text x="390" y="110" text-anchor="middle" fill="#232D4B">job 4821</text>
      <rect x="344" y="122" width="92" height="20" rx="3" fill="#ffffff" stroke="#232D4B" stroke-width="1"/>
      <text x="390" y="136" text-anchor="middle" fill="#232D4B">job 4822</text>
      <rect x="344" y="148" width="92" height="20" rx="3" fill="#E57200" fill-opacity="0.3" stroke="#E57200" stroke-width="1.5"/>
      <text x="390" y="162" text-anchor="middle" fill="#232D4B">yours</text>
    </g>
    <g stroke="#232D4B" stroke-width="2" fill="none">
      <path d="M450 125 L 492 91"/><path d="M450 125 L 492 139"/><path d="M450 125 L 492 187"/>
    </g>
    <g fill="#232D4B">
      <rect x="500" y="70" width="120" height="42" rx="5"/>
      <rect x="500" y="118" width="120" height="42" rx="5"/>
      <rect x="500" y="166" width="120" height="42" rx="5"/>
    </g>
    <g fill="#ffffff" font-size="11" text-anchor="middle">
      <text x="560" y="95">compute node</text>
      <text x="560" y="143">compute node</text>
      <text x="560" y="191">compute node</text>
    </g>
    <text x="320" y="236" text-anchor="middle" fill="#6b7280" font-size="12">You don't start the work. You describe it, hand it over, and wait your turn.</text>
  </g>
</svg>

What you get in exchange is scale you could never buy yourself, at no cost to
you, with software already installed and a filesystem visible from every node.
Running two hundred variations of a model at once is exactly what this machine
is for.

What it costs you is control and immediacy. Your job waits in line, and asking
for more resources or longer runtimes usually means waiting longer, so requesting
64 cores when your code uses one is both wasteful and slow for you. Jobs have
hard time limits and get killed at the limit whether or not they were nearly
done, which makes checkpointing a habit worth forming early. You have no root,
so software comes from environment modules, Conda, or containers rather than
`apt install`. And it's a batch world: you submit and come back later, which is
a different rhythm from a notebook.

Use it for: long runs, many runs at once, anything needing more memory than you
can rent comfortably, and anything you'd rather not pay for.

---

## Picking one

| Situation | Where |
|:---|:---|
| Writing and debugging code | Laptop |
| Data fits in memory and the job takes minutes | Laptop |
| One specific machine shape, right now, for a few hours | Cloud |
| Data already sits in a cloud bucket | Cloud |
| A current datacenter GPU, on demand | Cloud |
| Hundreds of independent runs | HPC |
| Days of runtime, or more RAM than you want to pay for | HPC |
| A grant or course already covers the cluster | HPC |

{: .note }
Before you ask for more of anything, find out what you actually used. `nproc`
and `htop` show cores and whether you're using them, `free -h` shows memory,
`nvidia-smi` shows GPU use and VRAM, and `df -h` shows disk. On Linux,
`/usr/bin/time -v python script.py` prints peak memory for the whole run, and on
a cluster `sacct -j <jobid> --format=JobID,MaxRSS,Elapsed` tells you the same
thing after the fact. Most requests for a bigger machine turn out to be requests
for one core and 3 GB.

The path most projects take is all three in sequence. Write it on your laptop
against a sample, confirm it works, then move the full run to whichever of the
cluster or the cloud is cheaper and less annoying for that particular job. The
code should not have to change much between them, and if it does, that's usually
worth fixing before you scale up.

---

## Going further

- [UVA Research Computing](https://www.rc.virginia.edu/) for cluster accounts,
  partitions, and the storage rules that apply here.
- [Slurm quick reference](https://slurm.schedmd.com/quickstart.html)
- [EC2 instance types](https://aws.amazon.com/ec2/instance-types/), which is
  also a readable tour of how machines get sized.
- [Storage]({{ site.baseurl }}/docs/storage.html) for what to do with the data
  once the job finishes.
