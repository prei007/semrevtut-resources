# From study coding to to reusable synthesis data

This repository accompanies the tutorial **'From study coding to reusable synthesis data: A semantic workflow for handover and reuse'**.

The tutorial begins with the kind of carefully designed, multi-table coding form described by Nelson et al. (2025). It asks what happens at handover: how another review team can inspect, extend, and selectively reuse earlier coding without having to reconstruct all of its meaning from workbook structure and local documentation.

The worked example is a fictional review of intervention conditions. It uses two familiar coding dimensions:

- intervention content domain, with values such as *Fractions*, *Algebra*, and *Word-problem solving*; and
- implementation-fidelity monitoring, distinguishing *Yes*, *No*, *Not reported*, and *Not applicable*.

The tutorial develops four layers:

1.  RDF identifies publications, studies, conditions, coding items, and coding acts and connects them through explicit relations.
2.  Web Annotation connects a coded response to the publication or passage on which it is based.
3.  PROV-O records reviewers, coding activities, manual versions, disagreement, and adjudication.
4.  An optional SKOS extension gives selected codebook concepts stable identities, definitions, hierarchical relations, and mappings.

SPARQL queries retrieve conventional tables from these layers and check their structure. The standards make coding data more addressable and portable; they do not make a coding manual valid or a reviewer judgement correct.

## Repository contents

``` text
semrevtut/
├── studies/
│   ├── intervention-review-example.trig
│   └── intervention-review-skos-annotations.trig
├── code_books/
│   ├── content_domains.skos.ttl
│   ├── DDI_ModeOfCollection_5.0.1.skos.ttl
│   └── DDI_TypeOfInstrument_1.1.2.skos.ttl
├── queries/
│   ├── list-intervention-codings.rq
│   ├── find-intervention-disagreements.rq
│   ├── check-intervention-provenance.rq
│   └── list-mathematics-domain-codings.rq
```

The core TriG file is self-contained. The SKOS example is deliberately kept in companion files so readers can complete the RDF, Web Annotation, provenance, and SPARQL workflow without first adopting a semantic codebook.

## Requirements

Running the examples requires:

- Java 21 or later;
- the Apache Jena binary distribution, which contains the `arq` and `riot` command-line tools; and
- a terminal opened in this repository's `semrevtut` directory.
- 

## Install Apache Jena

### 1. Check Java

Apache Jena 6 requires Java 21 or later. Check the installed version:

``` bash
java -version
```

If Java is absent or older than version 21, install a current JDK through your operating system's package manager or a JDK distributor before continuing.

### 2. Download and extract Jena

Open the [official Apache Jena downloads page](https://jena.apache.org/download/) and download the current **Apache Jena binary distribution** named `apache-jena-VERSION.zip` or `apache-jena-VERSION.tar.gz`. The separate Fuseki download is a SPARQL server and is not required for these local examples.

Extract the archive to a stable directory. The extracted directory contains `bin/` scripts for macOS and Linux and `bat/` scripts for Windows.

### 3. Put the tools on the command path

For macOS or Linux, set `JENA_HOME` to the extracted directory and add its `bin` directory to `PATH`:

``` bash
export JENA_HOME="/path/to/apache-jena-VERSION"
export PATH="$PATH:$JENA_HOME/bin"
```

Add these two lines to the configuration file used by your shell if you want the setting to persist in new terminal sessions.

For Windows PowerShell, use:

``` powershell
$env:JENA_HOME = "C:\path\to\apache-jena-VERSION"
$env:Path += ";$env:JENA_HOME\bat"
```

Windows command names may need the `.bat` suffix, for example `arq.bat` and `riot.bat`.

### 4. Verify the installation

``` bash
arq --version
riot --version
```

The [Jena command-line tools documentation](https://jena.apache.org/documentation/tools/) provides additional platform-specific setup and troubleshooting information.

## Run the core examples

Run all commands from the `semrevtut` directory.

### Retrieve the accepted coding table

``` bash
arq --data=studies/intervention-review-example.trig \
    --query=queries/list-intervention-codings.rq
```

Expected result: three rows connecting reports, studies, and intervention conditions to their accepted content-domain and fidelity-monitoring responses.

### Find the pre-adjudication coding difference

``` bash
arq --data=studies/intervention-review-example.trig \
    --query=queries/find-intervention-disagreements.rq
```

Expected result: one record for condition `C01`. Reviewer 1 assigned *Yes* and Reviewer 2 assigned *Not reported* before adjudication.

### Check the small provenance profile

``` bash
arq --data=studies/intervention-review-example.trig \
    --query=queries/check-intervention-provenance.rq
```

Expected result: no rows. An empty result means that every annotation has the fields required by the tutorial's profile. It does not establish that the coding decisions are substantively correct.

## Run the SKOS hierarchy example

The optional extension replaces textual content-domain bodies with references to concepts in a small SKOS scheme. Its hierarchy places specific codes below broader mathematical domains.

``` bash
arq --data=studies/intervention-review-example.trig \
    --data=studies/intervention-review-skos-annotations.trig \
    --data=code_books/content_domains.skos.ttl \
    --query=queries/list-mathematics-domain-codings.rq
```

Expected result:

| Condition | Specific domain      | Immediate broader domain |
|-----------|----------------------|--------------------------|
| `C01`     | Fractions            | Number and operations    |
| `C02`     | Algebra              | Mathematics              |
| `C03`     | Word-problem solving | Mathematics              |

All three records match the query for concepts at or below *Mathematics*, while their more specific original codes remain available.

## Validate the RDF files

`riot` can check that the Turtle and TriG syntax is valid:

``` bash
riot --validate studies/intervention-review-example.trig
riot --validate studies/intervention-review-skos-annotations.trig
riot --validate code_books/content_domains.skos.ttl
```

Successful validation produces no error message.

## Representation standards

- **RDF** is the underlying graph data model, written here as Turtle and TriG.
- **Web Annotation** represents identifiable coding acts with bodies and evidence targets.
- **PROV-O** represents agents, activities, attribution, generation, use, and derivation.
- **SKOS** represents the optional concept scheme, including labels, definitions, notes, hierarchies, and mappings.
- **SPARQL** retrieves, joins, and checks the resulting graph data.

The examples use fictional reports and demonstration metadata. Source publications are not distributed in this repository.

## Licence and citation

The project is licensed under the [GNU General Public License v3](LICENSE).
