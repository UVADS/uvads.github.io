---
title: Working with Databases
layout: default
nav_order: 6
description: Relational databases, NoSQL databases, and DuckDB, what each is actually for, and how to choose for a data science project.
---

# Database Basics
{: .fs-9 }

Sooner or later a folder of CSV files stops working. Two people need the data at
once, or a question comes up that pandas can only answer by reading 12 GB from
disk. That is when you want a database. The first fork in the road is relational
versus NoSQL, and for most coursework and most research projects the answer is
relational.
{: .fs-6 .fw-300 }

---

## Relational databases

A relational database stores tables. You decide the columns before you load any
data, you say what type each column holds, and from then on every row in that
table has exactly those columns. A `students` table has a `student_id` and a
`name` for all one thousand rows, or the database refuses the write.

The word "relational" trips people up. It isn't about relationships in the loose
sense of things being connected. Tables are linked by matching values: your
`enrollments` table stores the number 1001 in its `student_id` column, your
`students` table stores 1001 in its own `student_id` column, and a `JOIN` lines
them up when you ask. Nothing is stored twice and no pointers are involved.

Two you'll meet constantly. **MySQL** sits under an enormous number of web
applications and is the default on most cheap hosting. **PostgreSQL** is what
most data teams pick now, because it keeps absorbing features you eventually
want: window functions for ranking and running totals, `JSONB` columns for when
part of your data really is shapeless, PostGIS for maps, and pgvector for
embeddings.

<svg viewBox="0 0 640 250" width="100%" style="max-width:640px;height:auto;display:block;margin:1.5rem auto;" role="img" aria-labelledby="rel-title">
  <title id="rel-title">Two tables, students and enrollments, linked by matching student_id values</title>
  <g font-family="system-ui,sans-serif">
    <text x="30" y="42" fill="#232D4B" font-size="13" font-weight="700">students</text>
    <rect x="30" y="52" width="230" height="28" fill="#232D4B"/>
    <text x="44" y="71" fill="#ffffff" font-size="11.5" font-weight="600">student_id</text>
    <text x="160" y="71" fill="#ffffff" font-size="11.5" font-weight="600">name</text>
    <rect x="30" y="80" width="230" height="28" fill="#dbe3f0" stroke="#232D4B" stroke-width="1"/>
    <text x="44" y="99" fill="#232D4B" font-size="11.5" font-family="ui-monospace,monospace">1001</text>
    <text x="160" y="99" fill="#232D4B" font-size="11.5" font-family="ui-monospace,monospace">Ada</text>
    <rect x="30" y="108" width="230" height="28" fill="#ffffff" stroke="#232D4B" stroke-width="1"/>
    <text x="44" y="127" fill="#232D4B" font-size="11.5" font-family="ui-monospace,monospace">1002</text>
    <text x="160" y="127" fill="#232D4B" font-size="11.5" font-family="ui-monospace,monospace">Ken</text>
    <rect x="30" y="52" width="230" height="84" fill="none" stroke="#232D4B" stroke-width="2"/>
    <text x="370" y="42" fill="#232D4B" font-size="13" font-weight="700">enrollments</text>
    <rect x="370" y="52" width="240" height="28" fill="#232D4B"/>
    <text x="384" y="71" fill="#ffffff" font-size="11.5" font-weight="600">student_id</text>
    <text x="490" y="71" fill="#ffffff" font-size="11.5" font-weight="600">course</text>
    <rect x="370" y="80" width="240" height="28" fill="#dbe3f0" stroke="#232D4B" stroke-width="1"/>
    <text x="384" y="99" fill="#232D4B" font-size="11.5" font-family="ui-monospace,monospace">1001</text>
    <text x="490" y="99" fill="#232D4B" font-size="11.5" font-family="ui-monospace,monospace">DS 5100</text>
    <rect x="370" y="108" width="240" height="28" fill="#ffffff" stroke="#232D4B" stroke-width="1"/>
    <text x="384" y="127" fill="#232D4B" font-size="11.5" font-family="ui-monospace,monospace">1002</text>
    <text x="490" y="127" fill="#232D4B" font-size="11.5" font-family="ui-monospace,monospace">DS 5001</text>
    <rect x="370" y="136" width="240" height="28" fill="#dbe3f0" stroke="#232D4B" stroke-width="1"/>
    <text x="384" y="155" fill="#232D4B" font-size="11.5" font-family="ui-monospace,monospace">1001</text>
    <text x="490" y="155" fill="#232D4B" font-size="11.5" font-family="ui-monospace,monospace">DS 6013</text>
    <rect x="370" y="52" width="240" height="112" fill="none" stroke="#232D4B" stroke-width="2"/>
    <g stroke="#E57200" stroke-width="2.5" fill="none">
      <path d="M260 94 L 370 94"/>
      <path d="M260 94 C 310 94, 320 150, 370 150"/>
    </g>
    <circle cx="260" cy="94" r="4" fill="#E57200"/>
    <text x="315" y="84" text-anchor="middle" fill="#E57200" font-size="11" font-weight="600">student_id</text>
    <text x="315" y="188" text-anchor="middle" fill="#6b7280" font-size="11">1 student : many rows</text>
    <text x="320" y="215" text-anchor="middle" fill="#5b6478" font-size="11" font-family="ui-monospace,monospace">SELECT name, course FROM students JOIN enrollments USING (student_id)</text>
    <text x="320" y="238" text-anchor="middle" fill="#6b7280" font-size="12">Every row has the same columns. Tables connect by matching values.</text>
  </g>
</svg>

Two ideas do most of the work here. A **transaction** is a group of changes that
either all happen or none of them do; if the server loses power halfway through,
you never find half the work applied. A **constraint** is a rule the database
enforces on every write, such as "`student_id` must be unique" or "`gpa` cannot
be negative." Constraints apply to everyone, including the teammate who didn't
read your README and the script you wrote at midnight.

### What it's good at

- **SQL transfers everywhere.** The same `SELECT` you learn on Postgres works on
  MySQL, SQLite, DuckDB, Snowflake, BigQuery, and Spark, with small dialect
  differences. Every BI tool speaks it, and `pandas.read_sql()` turns a query
  straight into a DataFrame. Few skills in this field pay off across as many
  tools.
- **Joins mean you store each fact once.** An advisor's name lives in one row of
  one table. Correct a typo there and every query that touches it is correct
  from that moment on.
- **The database refuses bad data.** Types and constraints catch the duplicate
  student, the negative age, and the string that wandered into a numeric column,
  at write time rather than three weeks later in your analysis.
- **Indexes.** A query that would scan fifty million rows can come back in
  milliseconds once you index the column you filter on. You add the index later,
  without changing the data.
- **Transactions** make multi-step updates safe, which matters as soon as more
  than one process writes.

### Where it hurts

- You define the schema before you load anything, and changing it later is a
  migration you have to plan, test, and run in step with your code. On a large
  live table that is genuine work.
- Data that genuinely varies in shape fits badly. A survey where each respondent
  answers a different subset of 200 questions turns into a table of mostly empty
  columns.
- Writes scale up rather than out. You get one primary server, and the usual
  answers are a bigger machine or read replicas. Splitting writes across
  machines is real engineering.
- Deeply nested data has to be flattened across several tables and rebuilt with
  joins on the way out. Postgres `JSONB` softens this, though at that point you
  are doing document storage inside a relational database.
- You have to learn SQL. It's worth it, but it is a real thing to learn on top
  of the Python you already know.

---

## NoSQL databases

NoSQL is not one kind of database. It's a label that stuck to a group of systems
built in the late 2000s whose only shared trait was not being relational. The
two you're most likely to touch, MongoDB and Redis, have almost nothing in
common with each other, so it's worth taking them separately.

### MongoDB, a document store

MongoDB stores documents, which for practical purposes are JSON objects. A
collection is a pile of documents, and two documents in the same collection do
not have to carry the same fields. If you're pulling from an API that hands back
JSON, you can store what it gave you without designing anything first.

<svg viewBox="0 0 640 290" width="100%" style="max-width:640px;height:auto;display:block;margin:1.5rem auto;" role="img" aria-labelledby="doc-title">
  <title id="doc-title">One MongoDB collection holding two documents with different fields</title>
  <g font-family="system-ui,sans-serif">
    <text x="30" y="40" fill="#232D4B" font-size="13" font-weight="700">collection: students</text>
    <rect x="30" y="50" width="580" height="200" rx="8" fill="none" stroke="#232D4B" stroke-width="2"/>
    <rect x="52" y="72" width="250" height="118" rx="6" fill="#dbe3f0" stroke="#232D4B" stroke-width="1.5"/>
    <g font-family="ui-monospace,monospace" font-size="11.5" fill="#232D4B">
      <text x="66" y="94">{</text>
      <text x="66" y="112">  "_id": 1001,</text>
      <text x="66" y="130">  "name": "Ada",</text>
      <text x="66" y="148">  "major": "DS"</text>
      <text x="66" y="166">}</text>
    </g>
    <rect x="330" y="72" width="258" height="156" rx="6" fill="#dbe3f0" stroke="#232D4B" stroke-width="1.5"/>
    <g font-family="ui-monospace,monospace" font-size="11.5" fill="#232D4B">
      <text x="344" y="94">{</text>
      <text x="344" y="112">  "_id": 1002,</text>
      <text x="344" y="130">  "name": "Ken",</text>
      <text x="344" y="148">  "major": "DS",</text>
      <text x="344" y="202">}</text>
    </g>
    <g font-family="ui-monospace,monospace" font-size="11.5" fill="#E57200">
      <text x="344" y="166">  "courses": ["DS 5100"],</text>
      <text x="344" y="184">  "advisor": "Lee"</text>
    </g>
    <line x1="336" y1="154" x2="336" y2="190" stroke="#E57200" stroke-width="3"/>
    <text x="320" y="272" text-anchor="middle" fill="#6b7280" font-size="12">Same collection, different fields. Nothing had to be declared first.</text>
  </g>
</svg>

**What it's good at.** Records whose shape varies are no trouble, and adding a
field to new documents takes no migration. What you store looks like what your
Python code already holds, so a dict goes in and a dict comes out with no
translation layer in between. Fetching one whole record by its id is fast, since
the entire thing sits in one place. It also shards across machines more readily
than a relational database does.

**Where it hurts.** Joins are an afterthought; `$lookup` exists and it is not
where Mongo is comfortable. Nothing stops your data from drifting, so one
document ends up with `"price": 10` and the next with `"price": "10"`, and the
only thing standing between you and that mess is your own application code.
Duplication bites: if you copy an advisor's name into 4,000 student documents
and the advisor changes their name, you now own an update script. The
aggregation pipeline is a genuinely capable query language, but it's a second
one to learn and it doesn't transfer anywhere else the way SQL does. The pattern
to watch for is storing data quickly now and finding it hard to query later.

### Redis, a key-value store in memory

Redis keeps its data in RAM. You hand it a key, it hands back a value. The
lookup itself takes microseconds, and even across the network you're usually
under a millisecond. It also understands a few useful shapes beyond plain
strings: counters, lists, sets, and sorted sets, which is why leaderboards and
rate limiters get built on it constantly.

Redis is rarely the place your data lives. It sits in front of something else as
a cache, holds login sessions, counts requests per user, or passes jobs between
processes as a queue. Set an expiry on a key and it cleans itself up.

<svg viewBox="0 0 640 260" width="100%" style="max-width:640px;height:auto;display:block;margin:1.5rem auto;" role="img" aria-labelledby="kv-title">
  <title id="kv-title">A Redis instance holding three keys in memory, with an optional snapshot to disk</title>
  <g font-family="system-ui,sans-serif">
    <text x="320" y="32" text-anchor="middle" fill="#232D4B" font-size="14" font-weight="700">Redis, everything held in RAM</text>
    <rect x="90" y="44" width="460" height="132" rx="8" fill="none" stroke="#232D4B" stroke-width="2"/>
    <text x="118" y="63" fill="#6b7280" font-size="10.5">key</text>
    <text x="300" y="63" fill="#6b7280" font-size="10.5">value</text>
    <text x="466" y="63" fill="#6b7280" font-size="10.5">expires</text>
    <rect x="106" y="70" width="428" height="28" rx="4" fill="#dbe3f0" stroke="#232D4B" stroke-width="1"/>
    <rect x="106" y="106" width="428" height="28" rx="4" fill="#ffffff" stroke="#232D4B" stroke-width="1"/>
    <rect x="106" y="142" width="428" height="28" rx="4" fill="#dbe3f0" stroke="#232D4B" stroke-width="1"/>
    <g font-family="ui-monospace,monospace" font-size="11.5">
      <text x="118" y="89" fill="#232D4B" font-weight="600">session:8f2a</text>
      <text x="300" y="89" fill="#5b6478">{"user": 1001}</text>
      <text x="466" y="89" fill="#E57200">30 min</text>
      <text x="118" y="125" fill="#232D4B" font-weight="600">rate:1001</text>
      <text x="300" y="125" fill="#5b6478">47</text>
      <text x="466" y="125" fill="#E57200">60 sec</text>
      <text x="118" y="161" fill="#232D4B" font-weight="600">leaderboard</text>
      <text x="300" y="161" fill="#5b6478">sorted set, 10 items</text>
      <text x="466" y="161" fill="#6b7280">never</text>
    </g>
    <line x1="320" y1="176" x2="320" y2="194" stroke="#232D4B" stroke-width="2" stroke-dasharray="5 4"/>
    <polygon points="314,194 326,194 320,202" fill="#232D4B"/>
    <rect x="220" y="204" width="200" height="28" rx="6" fill="#ffffff" stroke="#9aa5bb" stroke-width="1.5" stroke-dasharray="4 3"/>
    <text x="320" y="223" text-anchor="middle" fill="#6b7280" font-size="11">optional snapshot to disk</text>
    <text x="320" y="252" text-anchor="middle" fill="#6b7280" font-size="12">One key, one lookup. Restart without saving and it's empty.</text>
  </g>
</svg>

**What it's good at.** Speed, mostly, and the fact that there is very little to
learn: `SET`, `GET`, `INCR`, `EXPIRE` and you're productive. Putting it in front
of Postgres can take a heavily repeated query off the database entirely. Expiry
is built in, so caches and sessions clean up after themselves.

**Where it hurts.** Your data has to fit in RAM, and RAM is the expensive part
of any server. Durability is optional and off by default in some setups, so
treat anything in Redis as losable unless you have deliberately configured
snapshots or the append-only log. There is no query language: you can fetch by
key, and that's the deal. You cannot ask it for every user in Virginia, because
the values are opaque to it. And a cache is another moving piece that can serve
you stale answers, which is its own category of confusing bug.

---

## DuckDB, the one that reads your files where they sit

DuckDB belongs in neither box above, and it's worth meeting early because it
deletes a step you have probably been doing by hand for years.

It is a real relational database: SQL, tables, joins, window functions, all of
it. It is also embedded the way SQLite is, so there's no server, no port, no
user accounts, and no daemon to remember to start. `pip install duckdb` and
you're done. The difference from SQLite is what it's tuned for. SQLite is built
for many small reads and writes of individual rows; DuckDB stores data by column
and processes it in batches, which is what makes it tear through aggregate
queries over millions of rows on an ordinary laptop.

The part that changes how you work is that it queries data files directly.
Nothing gets loaded or imported first.

```sql
SELECT species, avg(body_mass_g)
FROM 'penguins.csv'
GROUP BY species;
```

That is the entire program. No `CREATE TABLE`, no import step, no waiting.
Replace the filename with `'data/*.parquet'` and it reads a whole directory as
one table. Point it at `'s3://my-bucket/year=2026/*.parquet'` and it reads
straight from object storage, fetching only the columns and row groups your
query actually touches rather than the whole file. On a 40 GB Parquet dataset
where you asked for two columns and one date range, that is the difference
between minutes and hours, and between a small egress bill and a memorable one.

<svg viewBox="0 0 640 280" width="100%" style="max-width:640px;height:auto;display:block;margin:1.5rem auto;" role="img" aria-labelledby="duck-title">
  <title id="duck-title">DuckDB driven from a CLI, Python, R or a notebook, reading CSV, Parquet and JSON files locally, in S3, and in other databases</title>
  <g font-family="system-ui,sans-serif">
    <text x="69" y="44" text-anchor="middle" fill="#6b7280" font-size="10.5">drive it from</text>
    <g font-size="11.5">
      <rect x="14" y="58" width="110" height="36" rx="6" fill="#ffffff" stroke="#232D4B" stroke-width="2"/>
      <text x="69" y="81" text-anchor="middle" fill="#232D4B">duckdb CLI</text>
      <rect x="14" y="104" width="110" height="36" rx="6" fill="#ffffff" stroke="#232D4B" stroke-width="2"/>
      <text x="69" y="127" text-anchor="middle" fill="#232D4B">Python</text>
      <rect x="14" y="150" width="110" height="36" rx="6" fill="#ffffff" stroke="#232D4B" stroke-width="2"/>
      <text x="69" y="173" text-anchor="middle" fill="#232D4B">R</text>
      <rect x="14" y="196" width="110" height="36" rx="6" fill="#ffffff" stroke="#232D4B" stroke-width="2"/>
      <text x="69" y="219" text-anchor="middle" fill="#232D4B">notebook</text>
    </g>
    <g stroke="#E57200" stroke-width="2.5" fill="none">
      <path d="M124 76 C 170 76, 170 150, 210 150"/>
      <path d="M124 122 C 170 122, 170 150, 210 150"/>
      <path d="M124 168 C 170 168, 170 150, 210 150"/>
      <path d="M124 214 C 170 214, 170 150, 210 150"/>
    </g>
    <rect x="210" y="95" width="180" height="110" rx="8" fill="#232D4B"/>
    <text x="300" y="128" text-anchor="middle" fill="#ffffff" font-size="16" font-weight="700">DuckDB</text>
    <text x="300" y="150" text-anchor="middle" fill="#c9d1e3" font-size="11">one binary, no server</text>
    <text x="300" y="168" text-anchor="middle" fill="#c9d1e3" font-size="11">SQL in, DataFrame out</text>
    <text x="300" y="186" text-anchor="middle" fill="#c9d1e3" font-size="11">reads only what it needs</text>
    <g stroke="#E57200" stroke-width="2.5" fill="none">
      <path d="M390 150 C 430 150, 430 68, 470 68"/>
      <path d="M390 150 C 430 150, 430 114, 470 114"/>
      <path d="M390 150 C 430 150, 430 160, 470 160"/>
      <path d="M390 150 C 430 150, 430 206, 470 206"/>
    </g>
    <text x="548" y="40" text-anchor="middle" fill="#6b7280" font-size="10.5">it reads, in place</text>
    <g font-family="ui-monospace,monospace" font-size="10.5">
      <rect x="470" y="50" width="156" height="36" rx="6" fill="#dbe3f0" stroke="#232D4B" stroke-width="1.5"/>
      <text x="548" y="73" text-anchor="middle" fill="#232D4B">data.csv, data.json</text>
      <rect x="470" y="96" width="156" height="36" rx="6" fill="#dbe3f0" stroke="#232D4B" stroke-width="1.5"/>
      <text x="548" y="119" text-anchor="middle" fill="#232D4B">data.parquet</text>
      <rect x="470" y="142" width="156" height="36" rx="6" fill="#dbe3f0" stroke="#232D4B" stroke-width="1.5"/>
      <text x="548" y="165" text-anchor="middle" fill="#232D4B">s3://bucket/*.parquet</text>
      <rect x="470" y="188" width="156" height="36" rx="6" fill="#dbe3f0" stroke="#232D4B" stroke-width="1.5"/>
      <text x="548" y="211" text-anchor="middle" fill="#232D4B">Postgres, SQLite</text>
    </g>
    <text x="320" y="262" text-anchor="middle" fill="#6b7280" font-size="12">The data never moves. One query can span a local file, a bucket, and a database.</text>
  </g>
</svg>

### Poking around by hand

Run `duckdb` with no arguments and you get a shell that prints proper boxed
tables, so answering "what is actually in this file" takes one line instead of a
notebook:

```bash
duckdb -c "DESCRIBE SELECT * FROM 'readings.parquet'"
duckdb -c "SELECT count(*) FROM 's3://my-bucket/logs/*.json'"
duckdb -c "FROM 'readings.parquet' LIMIT 20"
```

That last one is not a typo. DuckDB lets you start a query with `FROM`, which
makes quick looks quicker. Recent versions also ship a local browser interface
with `duckdb -ui` if you'd rather click through a dataset than type at it.

### The same thing from code

```python
import duckdb

df = duckdb.sql("""
    SELECT station, date_trunc('day', ts) AS day, max(temp_c)
    FROM 's3://my-bucket/readings/*.parquet'
    WHERE station = 'CHO'
    GROUP BY 1, 2
""").df()
```

`.df()` hands back a pandas DataFrame, and `.arrow()` or `.pl()` give you Arrow
or Polars instead. It works in the other direction too: a DataFrame sitting in
memory is queryable by its variable name, so you can drop into SQL for the one
join that's awkward in pandas and come straight back out, without writing
anything to disk.

```python
import pandas as pd, duckdb

readings = pd.read_csv("readings.csv")
duckdb.sql("SELECT station, max(temp_c) FROM readings GROUP BY station").df()
```

The connector part goes further than files. DuckDB can `ATTACH` a running
Postgres, MySQL, or SQLite database and query it as though its tables were
local, which means one statement can join a Parquet file in a bucket against a
table in your department's Postgres. That is usually the moment people stop
thinking of it as a database and start thinking of it as the thing that talks to
everything else.

**Where it hurts.** It's built for one process at a time. Several readers are
fine, but it is not an application backend and it will not serve concurrent web
traffic, so Postgres keeps that job. It runs in memory unless you hand it a file
to persist to, and while it spills to disk for many operations that exceed RAM,
not every operation does. It also moves fast as a project, having only reached
1.0 in 2024, so pin your version if you care about reproducibility.

---

## Picking one

| Situation | Use |
|:---|:---|
| Questions about files you already have, local or in a bucket | DuckDB |
| Analytics over millions of rows on one machine | DuckDB |
| Records that all share the same fields | Relational |
| You need to combine data from several places in one query | Relational |
| Correctness matters more than raw write speed | Relational |
| Anyone downstream will use SQL or a BI tool | Relational |
| JSON from an API, shape varies per record | MongoDB |
| Each record is read and written whole, and rarely joined | MongoDB |
| A cache in front of a slower database | Redis |
| Counters, sessions, rate limits, queues, leaderboards | Redis |

{: .note }
For a single-user project on your laptop, reach for DuckDB before you set up any
server at all. There is nothing to install beyond a `pip install`, nothing to
run, and it speaks ordinary SQL, so the queries carry over if you outgrow it and
move to Postgres later. Plenty of analysis work never needs more than this.

Real systems commonly run two of these at once, and the split is usually the
same: Postgres holds the authoritative data, and Redis holds a fast copy of
whatever gets read over and over. Start with the relational database. Add
something else when you can name the specific problem it solves.

---

## Going further

- [PostgreSQL tutorial](https://www.postgresql.org/docs/current/tutorial.html)
- [MySQL documentation](https://dev.mysql.com/doc/)
- [MongoDB manual](https://www.mongodb.com/docs/manual/)
- [Redis documentation](https://redis.io/docs/latest/)
- [DuckDB documentation](https://duckdb.org/docs/), and its
  [data import guide](https://duckdb.org/docs/stable/data/overview) for reading
  files and buckets directly.
