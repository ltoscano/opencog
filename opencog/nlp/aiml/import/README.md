AIML to Atomese Conversion
--------------------------
This directory contains a perl script to convert AIML to OpenCog
Atomese, so that the AIML rules can be integrated into the rest of
the OpenCog natural language processing infrastructure.

The current target of the importer is to generate OpenPsi-compatable
rules, so that the openpsi rule engine can act as an AIML engine.


### Usage
Convert all AIML into Atomese:
```
perl aiml2oc.pl --dir ./aimldir --out atomese.scm
```
where, `./aimldir` contains aiml files and `atomese.scm` is the output,
containing the atomese representation of aiml rules in scheme.

Note that standard AIML semantics is that only the final, last defintion
of a category is taken to be definitive, so that earlier definitions are
ignored/discarded.  This can be accomplished here, with the
`--last-only` option, like so:
```
perl aiml2oc.pl --last-only --dir ./aimldir --out atomese.scm
```

For details, run:
```
perl aiml2oc.pl --help
```
For aiml files, see https://github.com/jstnhuang/chatbot/tree/master/aiml

### Overview
Conversion is done in a two-pass process.  The first pass flattens
the AIML format into a simplified linear format.  A second pass
converts this flattened format into Atomese.

* Pass One : The AIML XML is converted into an intermediate neutral,
  word based format. The format is a linear sequence of tokens,
  annotated with their meaning, and optionally any performative code.
  The AIML 2.0 interpreter uses something similar to this, with
  its AIMLIF csv based format.

* Pass Two: The linearized format is converted into Atomese.

One issue with conversion is the AIML convention that "the last
definition loaded is definitive".  Typical AIML systems will sort and
load AIML files in alphabetical order, so that the later files "overlay"
or "overwrite" previous definitions. By default, this AIML-to-opencog
converter makes all defintions visible to OpenCog, and so the OpenCog
NLP system can choos to decide how it wants to handle duplicate definitions.
Alternately, the `--last-only` option can be used to preserve the
standard AIML semantics, and discard all but the last definitions.

### Design notes
One reason for having an intermediate format is the handling of `<set>`
tags in patterns. These could either be passed on to OpenCog as either
a pointer to a collection concept, the collection list itself, a
collection concept with its definition, or various ways of unrolling
the defined set in intermediate format using duplicate definitions.

### Output format

The output format is in the form of OpenPsi rules, which have the format:
```
(Implication
	(AndLink  (stv strength confidence)
		(context)
		(action))
	(demand-goal))

(Member (action) (Concept "OpenPsi: action"))

(Member (implication) (demandgoal))
```

The scheme functions `psi-rule` and `psi-demand` create these.