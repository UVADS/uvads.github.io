---
title: Working with Databases
layout: default
nav_order: 6
description: Relational databases and NoSQL databases, what each is actually for, and how to choose for a data science project.
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

## Picking one

| Situation | Use |
|:---|:---|
| Records that all share the same fields | Relational |
| You need to combine data from several places in one query | Relational |
| Correctness matters more than raw write speed | Relational |
| Anyone downstream will use SQL or a BI tool | Relational |
| JSON from an API, shape varies per record | MongoDB |
| Each record is read and written whole, and rarely joined | MongoDB |
| A cache in front of a slower database | Redis |
| Counters, sessions, rate limits, queues, leaderboards | Redis |

{: .note }
For a single-user project on your laptop, SQLite deserves a look before you set
up any server at all. It's one file, it ships with Python as `sqlite3`, there is
nothing to install or run, and it speaks ordinary SQL. Plenty of analysis work
never needs more, and moving to Postgres later is a small step because the
queries mostly carry over.

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
- [SQLite](https://www.sqlite.org/whentouse.html) on when it's the right choice,
  which is more often than people expect.
