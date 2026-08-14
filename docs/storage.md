---
title: Storage
layout: default
nav_order: 5
description: What block storage and object storage actually are, what each one costs you, and how to pick.
---

# Storage Basics
{: .fs-9 }

Every project eventually has to put its data somewhere. Nearly always the choice
is between block storage and object storage. From a distance they look like the
same thing with different billing; once you start writing to them they behave
nothing alike.
{: .fs-6 .fw-300 }

---

## Block Storage

Block storage gives you a disk. Not a metaphor for a disk, an actual block
device: a fixed-size range of addresses carved into chunks of 512 bytes or 4 KB,
with no concept of a file. Something else has to supply that concept, and that
something is a filesystem. You run `mkfs` once, ext4 or XFS or APFS writes its
own bookkeeping into the first blocks, and from then on the kernel translates
`/data/results.csv` into "blocks 88,412 through 88,530."

On AWS this is EBS. On GCP it's Persistent Disk. In a machine room it's whatever
the SAN hands out over iSCSI. The laptop you're reading this on has one soldered
to the board.

The detail that drives everything else is attachment. A block volume connects to
one machine, the way an external drive does. That machine mounts it and treats
it as a normal path. Detach it and you have an inert volume sitting in a single
availability zone, reachable by nothing.

<svg viewBox="0 0 640 250" width="100%" style="max-width:640px;height:auto;display:block;margin:1.5rem auto;" role="img" aria-labelledby="block-title">
  <title id="block-title">A single virtual machine attached to one block volume made of fixed-size blocks</title>
  <rect x="20" y="60" width="170" height="120" rx="8" fill="#232D4B"/>
  <text x="105" y="95" text-anchor="middle" fill="#ffffff" font-family="system-ui,sans-serif" font-size="15" font-weight="600">Virtual Machine</text>
  <text x="105" y="120" text-anchor="middle" fill="#c9d1e3" font-family="system-ui,sans-serif" font-size="12">filesystem (ext4)</text>
  <text x="105" y="140" text-anchor="middle" fill="#c9d1e3" font-family="system-ui,sans-serif" font-size="12">mounted at /data</text>
  <line x1="190" y1="120" x2="290" y2="120" stroke="#E57200" stroke-width="4"/>
  <circle cx="290" cy="120" r="6" fill="#E57200"/>
  <text x="240" y="108" text-anchor="middle" fill="#E57200" font-family="system-ui,sans-serif" font-size="12" font-weight="600">attached</text>
  <text x="240" y="146" text-anchor="middle" fill="#6b7280" font-family="system-ui,sans-serif" font-size="11">1 volume : 1 host</text>
  <rect x="300" y="45" width="320" height="150" rx="8" fill="none" stroke="#232D4B" stroke-width="2"/>
  <text x="460" y="35" text-anchor="middle" fill="#232D4B" font-family="system-ui,sans-serif" font-size="14" font-weight="600">Block Volume (fixed size)</text>
  <g fill="#dbe3f0" stroke="#232D4B" stroke-width="1.5">
    <rect x="316" y="65" width="44" height="34" rx="3"/><rect x="368" y="65" width="44" height="34" rx="3"/>
    <rect x="420" y="65" width="44" height="34" rx="3"/><rect x="472" y="65" width="44" height="34" rx="3"/>
    <rect x="524" y="65" width="44" height="34" rx="3"/><rect x="560" y="65" width="44" height="34" rx="3" fill="#ffffff"/>
    <rect x="316" y="107" width="44" height="34" rx="3"/><rect x="368" y="107" width="44" height="34" rx="3" fill="#E57200" fill-opacity="0.35"/>
    <rect x="420" y="107" width="44" height="34" rx="3"/><rect x="472" y="107" width="44" height="34" rx="3" fill="#ffffff"/>
    <rect x="524" y="107" width="44" height="34" rx="3" fill="#ffffff"/><rect x="560" y="107" width="44" height="34" rx="3" fill="#ffffff"/>
    <rect x="316" y="149" width="44" height="34" rx="3" fill="#ffffff"/><rect x="368" y="149" width="44" height="34" rx="3" fill="#ffffff"/>
    <rect x="420" y="149" width="44" height="34" rx="3" fill="#ffffff"/><rect x="472" y="149" width="44" height="34" rx="3" fill="#ffffff"/>
    <rect x="524" y="149" width="44" height="34" rx="3" fill="#ffffff"/><rect x="560" y="149" width="44" height="34" rx="3" fill="#ffffff"/>
  </g>
  <text x="390" y="130" text-anchor="middle" fill="#232D4B" font-family="system-ui,sans-serif" font-size="10" font-weight="700">edit</text>
  <text x="460" y="225" text-anchor="middle" fill="#6b7280" font-family="system-ui,sans-serif" font-size="12">One byte changes, one 4 KB block gets rewritten</text>
</svg>

### What it's good at

- **Speed.** The volume sits on the host or one hop away on a dedicated storage
  network. A gp3 volume starts at 3,000 IOPS and 125 MB/s and can be dialed up
  to 16,000 IOPS. Latency is measured in microseconds and low milliseconds, not
  round trips.
- **Editing in place.** Change four bytes in the middle of a 40 GB file and the
  drive rewrites one block. Everything transactional depends on this. Postgres
  writes 8 KB pages and flushes a write-ahead log on every commit; if each of
  those commits meant rewriting the whole database file, the database would not
  work.
- **Programs already expect it.** File locks, `seek()`, `mmap`, atomic renames,
  permissions, symlinks. SQLite needs locking. Git needs renames. If a tool has
  ever asked you for a path, it wants a block device underneath.
- **Cheap snapshots**, because the service only stores the blocks that changed
  since the last one.

### Where it hurts

- One host at a time. Two machines that both need the data means running NFS or
  EFS or Lustre on top, which is a second system to operate.
- The size is a decision you make up front. Growing a volume means expanding the
  volume and then running `resize2fs` or `xfs_growfs` to tell the filesystem
  about the new room. Shrinking usually isn't possible at all.
- You're billed for what you allocated. A 1 TB volume holding 4 GB of data costs
  exactly what a full one costs.
- It lives in one availability zone. If the zone goes, so does the volume,
  unless you've been taking snapshots.
- Roughly $0.08 per GB-month for gp3 in us-east-1, against $0.023 for S3
  Standard. Check current prices before you quote those, but the ratio holds.
- Everything about it is yours to run: filesystem choice, backups, fsck after a
  bad shutdown, and the alert at 3 a.m. saying the disk is full.

---

## Object Storage

Object storage drops the disk idea entirely. There's no device to attach and no
filesystem to create. You get a bucket, and into it you put objects: a blob of
bytes, a key that identifies it, and whatever metadata you attach. All of it
moves over HTTPS with `PUT`, `GET`, and `DELETE`.

The namespace is flat. `aws s3 ls s3://my-bucket/raw/2026/` prints something
that looks like a directory listing, but nothing is nested; the service is
matching a string prefix against every key in the bucket, and the slashes are
ordinary characters that the console draws as folders because people find that
easier to read.

Objects are also immutable. There's no seek-and-write, no appending. Change one
byte in a 4 GB file and you upload 4 GB.

<svg viewBox="0 0 640 300" width="100%" style="max-width:640px;height:auto;display:block;margin:1.5rem auto;" role="img" aria-labelledby="object-title">
  <title id="object-title">Many clients reaching one bucket of keyed objects over an HTTP API, replicated across three availability zones</title>
  <g font-family="system-ui,sans-serif">
    <rect x="14" y="40" width="96" height="34" rx="6" fill="#ffffff" stroke="#232D4B" stroke-width="2"/>
    <text x="62" y="62" text-anchor="middle" fill="#232D4B" font-size="12">laptop</text>
    <rect x="14" y="92" width="96" height="34" rx="6" fill="#ffffff" stroke="#232D4B" stroke-width="2"/>
    <text x="62" y="114" text-anchor="middle" fill="#232D4B" font-size="12">web app</text>
    <rect x="14" y="144" width="96" height="34" rx="6" fill="#ffffff" stroke="#232D4B" stroke-width="2"/>
    <text x="62" y="166" text-anchor="middle" fill="#232D4B" font-size="12">Spark job</text>
    <rect x="14" y="196" width="96" height="34" rx="6" fill="#ffffff" stroke="#232D4B" stroke-width="2"/>
    <text x="62" y="218" text-anchor="middle" fill="#232D4B" font-size="12">CI pipeline</text>
    <g stroke="#E57200" stroke-width="2.5" fill="none">
      <path d="M110 57 C 150 57, 150 135, 186 135"/>
      <path d="M110 109 C 150 109, 150 135, 186 135"/>
      <path d="M110 161 C 150 161, 150 135, 186 135"/>
      <path d="M110 213 C 150 213, 150 135, 186 135"/>
    </g>
    <text x="150" y="26" text-anchor="middle" fill="#E57200" font-size="12" font-weight="600">GET / PUT over HTTPS</text>
    <rect x="190" y="40" width="290" height="190" rx="8" fill="none" stroke="#232D4B" stroke-width="2"/>
    <text x="335" y="30" text-anchor="middle" fill="#232D4B" font-size="14" font-weight="600">Bucket (flat, no size limit)</text>
    <g>
      <rect x="206" y="56" width="258" height="36" rx="5" fill="#dbe3f0" stroke="#232D4B" stroke-width="1.5"/>
      <text x="218" y="72" fill="#232D4B" font-size="11" font-weight="600">raw/2026/08/readings.parquet</text>
      <text x="218" y="86" fill="#5b6478" font-size="10">142 MB · etag · owner · storage-class</text>
      <rect x="206" y="100" width="258" height="36" rx="5" fill="#dbe3f0" stroke="#232D4B" stroke-width="1.5"/>
      <text x="218" y="116" fill="#232D4B" font-size="11" font-weight="600">models/v3/weights.safetensors</text>
      <text x="218" y="130" fill="#5b6478" font-size="10">4.1 GB · etag · owner · storage-class</text>
      <rect x="206" y="144" width="258" height="36" rx="5" fill="#dbe3f0" stroke="#232D4B" stroke-width="1.5"/>
      <text x="218" y="160" fill="#232D4B" font-size="11" font-weight="600">site/index.html</text>
      <text x="218" y="174" fill="#5b6478" font-size="10">8 KB · etag · owner · storage-class</text>
      <rect x="206" y="188" width="258" height="26" rx="5" fill="#ffffff" stroke="#9aa5bb" stroke-width="1.5" stroke-dasharray="4 3"/>
      <text x="335" y="205" text-anchor="middle" fill="#6b7280" font-size="11">billions more, same price per GB</text>
    </g>
    <g stroke="#232D4B" stroke-width="2" fill="none" stroke-dasharray="5 4">
      <path d="M480 90 L 520 70"/><path d="M480 135 L 520 135"/><path d="M480 180 L 520 200"/>
    </g>
    <g fill="#232D4B">
      <rect x="524" y="54" width="96" height="32" rx="5"/>
      <rect x="524" y="119" width="96" height="32" rx="5"/>
      <rect x="524" y="184" width="96" height="32" rx="5"/>
    </g>
    <g fill="#ffffff" font-size="12" text-anchor="middle">
      <text x="572" y="75">copy · AZ 1</text>
      <text x="572" y="140">copy · AZ 2</text>
      <text x="572" y="205">copy · AZ 3</text>
    </g>
    <text x="572" y="240" text-anchor="middle" fill="#6b7280" font-size="11">replicated for you</text>
  </g>
  <text x="320" y="272" text-anchor="middle" fill="#6b7280" font-family="system-ui,sans-serif" font-size="12">One byte changes, the whole object gets re-uploaded</text>
  <text x="320" y="290" text-anchor="middle" fill="#6b7280" font-family="system-ui,sans-serif" font-size="12">No host to attach to, no capacity to provision</text>
</svg>

### Why everyone copied S3

S3 shipped in March 2006, before EC2, and the API turned out to be small enough
that competitors implemented it rather than inventing their own. That is why
MinIO, Cloudflare R2, Backblaze B2, Wasabi, Ceph, and Google Cloud Storage all
answer S3 calls today, and why `boto3`, `rclone`, DuckDB, Spark, and pandas can
read from any of them after you point them at a different endpoint. Time spent
learning the S3 API is not time spent learning one vendor's product. The lock-in
is in the egress bill, not the interface.

### What it's good at

- **You never size it.** There is nothing to provision and nothing to grow. A
  bucket holding 3 KB and a bucket holding 3 PB are configured identically. The
  only ceiling worth knowing is 5 TB per object, and anything over 5 GB goes up
  as a multipart upload, which the CLI and the SDKs handle for you.
- **It survives things.** Every object is written to several separate facilities
  before the `PUT` returns. AWS quotes eleven nines of annual durability for S3
  Standard, and you get that number without configuring replication, RAID, or
  backups.
- **Nothing is attached to it.** The bucket has no host. A hundred machines
  across three regions can read the same object at the same time, and the data
  keeps existing after the cluster that produced it is torn down. That property
  is the reason data lakes are built this way.
- **Storing things is cheap enough to stop thinking about.** Around $0.023 per
  GB-month for Standard, plus fractions of a cent per thousand requests.
- **Lifecycle rules move old data down the price ladder** to Glacier and its
  colder tiers on a schedule you set once.
- Every object has a URL, so the same bucket can serve a static site, hand out
  a dataset, or accept uploads through presigned links.

### Where it hurts

- Latency is an HTTP round trip, tens to hundreds of milliseconds. Reading one
  200 MB Parquet file is fine. Reading 200,000 tiny files is miserable, and the
  fix is always the same: fewer, larger files.
- Immutability shapes your whole pipeline. You don't append a day of records to
  yesterday's file, you write today's file next to it and let the query engine
  read both.
- It is not a filesystem and won't pretend convincingly. No locks, no `seek()`,
  and a rename is a copy followed by a delete, which is why "moving" a large
  prefix takes real time and real money. `s3fs` and Mountpoint will mount a
  bucket for you, and the seams show under load.
- S3 has been strongly read-after-write consistent since 2020, but plenty of
  S3-compatible services still aren't. Verify before you build on it.
- Requests and egress are where the surprise bills come from. A job issuing
  millions of small `GET`s can spend more on requests than on storage, and data
  leaving the cloud runs about $0.09 per GB.
- Buckets are easy to expose. Public-by-accident buckets have leaked more
  records than most attacks have.

---

## Picking one

| Situation | Use |
|:---|:---|
| A database, or anything with a hot write path | Block |
| Software that expects an ordinary file path | Block |
| Heavy random reads and writes inside large files | Block |
| Raw data feeding analytics, a data lake, a warehouse staging area | Object |
| Backups, archives, logs, model checkpoints | Object |
| Anything many machines or many people read at once | Object |
| Data that has to outlive the machine that made it | Object |

{: .note }
Most real systems use both, in a specific arrangement: the bucket holds the
authoritative copy, and the volume holds a fast scratch copy. A training run
pulls its dataset from S3 onto a local disk, hammers that disk with random reads
for six hours, writes checkpoints back to S3, and then the instance and its
volume are deleted. Nothing of value lived on the volume.

If you're editing bytes where they sit, you want block. If you're writing whole
files once and reading them many times, you want object.

---

## Going further

- [Amazon S3 documentation](https://docs.aws.amazon.com/s3/)
- [Amazon EBS documentation](https://docs.aws.amazon.com/ebs/)
- [MinIO](https://min.io/) runs an S3-compatible server on your own machine,
  which is the cheapest way to practice the API without an AWS bill.
