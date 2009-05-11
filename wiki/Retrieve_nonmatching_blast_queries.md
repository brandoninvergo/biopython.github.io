---
title: Retrieve nonmatching blast queries
permalink: wiki/Retrieve_nonmatching_blast_queries
layout: wiki
tags:
 - Cookbook
---

Problem
-------

The XML output of NCBI's stand alone BLAST programs does not include
information on query sequences that have 'no hits' in the target
database. [Sometimes](http://bugzilla.open-bio.org/show_bug.cgi?id=2821)
you want to know which sequences don't have match a database and further
analyse/anotate them accordingly. There are a number of different ways
to do this, one is to use [SeqIO's method .to\_dict()](SeqIO "wikilink")
to turn the query file into a flat database then parse the results file
to get the sequences that *did* match the database. You can then use
python's set() arithmetic to make a list of sequences that are in the
query file and not the results which can be used as keys to retrieve the
complete [SeqRecord](SeqRecord "wikilink") for each of the "no hit"
queries. Got it? Well, perhaps it's easier to just do it:

Solution
--------

Let's presume you set up a BLAST run with the sequences in a file called
"queries.fasta" searched against a database, with the results saved to
BLAST\_RESULTS.xml

``` python
from Bio import SeqIO
from Bio.Blast import NCBIXML

q_dict =  SeqIO.to_dict(SeqIO.parse(open('queries.fasta', 'r'), 'fasta'))

# make our 'hit list', the 'query' in the BLAST output is the record's ''description'' while
# the key for our dictionary is the record id, we need to split the query and get the first
# field to get the right key. 
hits = []
for record in NCBIXML.parse(open("BLAST_RESULTS.xml", 'r')):
  hits.append(record.query.split()[0])
 
misses = set(q_dict.keys()) - set(hits)

orphans = []
for record in misses:
   orphans.append(q_dict[record])
```

We can do a litte sanity check to make sure everything worked OK:

``` python
>>> print "found %i records in query, %i have hits, making %i misses" % (len(q_dict.keys()), len(hits), len(misses))
>>> found 11955 records in query, 2802 have hits, making 9153 misses
```

Good, now you have all the 'not hits' sequence in a list ('orphans') of
SeqRecord objects you can annotate/analyse as you please or use [
SeqIO.write()](SeqIO "wikilink") to make a new file of just these
sequences that can be put through another program.

Discussion
----------

As implemented above most of the time in each run is spend populating
the list of hits from the BLAST parser, would checking each record from
the results file against the dictionary one at a time be a less memory
intensive way to go in case of very large files?