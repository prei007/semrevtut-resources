# From study coding to semantic review data

This repository accompanies the tutorial **“From study coding to semantic review data: A reproducible workflow using Web Annotation and SKOS.”** It shows how the products of study coding can be represented as explicit, interoperable, and reusable review data.

The tutorial is aimed at researchers and research students conducting qualitative, interpretive, theory-oriented, realist, configurative, mapping, or mixed-form syntheses. It begins **after searching and study selection**. It does not propose a new review methodology or replace familiar coding tools. Instead, it adds standards-based representation, provenance, quality-and-trust, and access layers to established review workflows.

Using a review of technology for nano-education as its running example, the tutorial moves cumulatively from a conventional coding table to a small, executable semantic review-data package:

1.  express coded relations as RDF subject–predicate–object statements;
2.  represent and reuse coding concepts with SKOS;
3.  anchor classifications and extracted claims to source passages with the Web Annotation Data Model;
4.  preserve reviewers, coding activities, codebook versions, decisions, and adjudication with a small PROV-O profile; and
5.  retrieve, check, and reshape the resulting data with SPARQL.

The workflow improves computational reproducibility, procedural reproducibility, interpretive traceability, and interoperability. It does **not** make a codebook valid, a passage representative, an interpretation correct, or a study finding credible. Those remain matters of review design, reviewer judgment, and methodological quality.

## Tutorial structure

The full tutorial is in [`manuscript-v1.qmd`](manuscript-v1.qmd). Its worked examples follow this progression:

| Stage | Main idea | Data and queries |
|----|----|----|
| Explicit relations | Transform a multi-column coding form into explicit RDF relations among publications, studies, learning activities, features, actions, outcomes, data, and claims. | [`studies/Millet2013_extracted.trig`](studies/Millet2013_extracted.trig) |
| Semantic codebooks | Give codes stable identities, labels, definitions, notes, and broader/narrower relations; reuse external concepts where they fit. | [`code_books/`](code_books/) and the vocabulary queries in [`queries/`](queries/) |
| Passage-anchored annotations | Represent a code assignment or extracted claim as an identifiable annotation connected to an exact source quotation and page. | [`studies/Millet2013_annotations.trig`](studies/Millet2013_annotations.trig), [`queries/list-millet-annotations.rq`](queries/list-millet-annotations.rq), and [`queries/list-millet-claim-annotations.rq`](queries/list-millet-claim-annotations.rq) |
| Auditable decisions | Retain two reviewers’ independent coding acts, agreement and disagreement, codebook/manual versions, statuses, and an adjudicated decision without deleting its inputs. | [`studies/Millet2013_provenance.trig`](studies/Millet2013_provenance.trig), [`queries/find-millet-coding-disagreements.rq`](queries/find-millet-coding-disagreements.rq), [`queries/list-millet-accepted-annotations.rq`](queries/list-millet-accepted-annotations.rq), and [`queries/check-millet-provenance.rq`](queries/check-millet-provenance.rq) |
| Cross-study retrieval | Apply the same annotation and provenance pattern across four publications, recreate a study-by-variable table, and query a broader SKOS concept family. | The Burgin, Höst, Lan, and Millet provenance files; [`queries/list-cross-study-codings.rq`](queries/list-cross-study-codings.rq); and [`queries/find-immersive-technology-studies.rq`](queries/find-immersive-technology-studies.rq) |

Millet et al. (2013), a study of haptics and graphic analogies for learning about atomic force microscopy, supplies the detailed running example. The final retrieval adds Burgin et al. (2020), Höst et al. (2013), and Lan and Azimi (2025), producing a comparable four-publication corpus.

## Requirements

The executable examples require:

- Java;
- [Apache Jena](https://jena.apache.org/), with the `arq` and `riot` commands on `PATH`; and
- a terminal opened in the repository root (`semrevtut`).

[Quarto](https://quarto.org/) is needed only to render the manuscript. The current PDF configuration also refers to a local APA CSL file; replace the absolute `csl` path in the manuscript front matter with a CSL file available on your system before rendering elsewhere.

## Run the examples

All commands below should be run from the `semrevtut` directory.

### Inspect passage-anchored annotations

``` bash
arq --data=studies/Millet2013_annotations.trig \
    --data=code_books/BloomsTaxonomy.skos.ttl \
    --query=queries/list-millet-annotations.rq
```

Expected result: three annotations—one Bloom classification and two extracted claims—with their motivations, body types, pages, and exact source passages.

To retrieve only the two evidence-backed claim annotations:

``` bash
arq --data=studies/Millet2013_annotations.trig \
    --query=queries/list-millet-claim-annotations.rq
```

### Inspect disagreement, accepted decisions, and provenance completeness

``` bash
arq --data=studies/Millet2013_provenance.trig \
    --data=code_books/BloomsTaxonomy.skos.ttl \
    --query=queries/find-millet-coding-disagreements.rq
```

Expected result: one disagreement on page 608, where two reviewers assigned *Analyze* and *Understand* to the same target.

``` bash
arq --data=studies/Millet2013_provenance.trig \
    --data=code_books/BloomsTaxonomy.skos.ttl \
    --data=code_books/research_methods.skos.ttl \
    --data=code_books/tech_features.skos.ttl \
    --query=queries/list-millet-accepted-annotations.rq
```

Expected result: the six accepted Millet coding records, including two agreeing reviewer acts and the jointly adjudicated Bloom classification.

Run the completeness check with the same four data files and [`queries/check-millet-provenance.rq`](queries/check-millet-provenance.rq):

``` bash
arq --data=studies/Millet2013_provenance.trig \
    --data=code_books/BloomsTaxonomy.skos.ttl \
    --data=code_books/research_methods.skos.ttl \
    --data=code_books/tech_features.skos.ttl \
    --query=queries/check-millet-provenance.rq
```

Expected result: no rows. An empty result means that every annotation satisfies the tutorial’s small provenance profile; it is not a general validation of the review data.

### Recreate the cross-study table

``` bash
arq --data=studies/Burgin2020_provenance.trig \
    --data=studies/Hoest2013_provenance.trig \
    --data=studies/Millet2013_provenance.trig \
    --data=studies/Lan2025_provenance.trig \
    --data=code_books/research_methods.skos.ttl \
    --data=code_books/tech_features.skos.ttl \
    --query=queries/list-cross-study-codings.rq
```

Expected result: four rows, one per publication, containing the accepted study-design and technology-feature codes, the responsible reviewers, and the codebook versions.

To retrieve accepted technologies in the *Immersive technologies* branch of the feature scheme:

``` bash
arq --data=studies/Burgin2020_provenance.trig \
    --data=studies/Hoest2013_provenance.trig \
    --data=studies/Millet2013_provenance.trig \
    --data=studies/Lan2025_provenance.trig \
    --data=code_books/tech_features.skos.ttl \
    --query=queries/find-immersive-technology-studies.rq
```

Expected result: Höst et al. (*Virtual reality*), Lan and Azimi (*Virtual laboratory*), and Millet et al. (*Scanning probe haptic interface*). Reviewer identities and timestamps are demonstration metadata.

## Repository contents

``` text
semrevtut/
├── manuscript-v1.qmd       Tutorial manuscript
├── references.bib          Bibliography used by the manuscript
├── code_books/             Local and reused SKOS concept schemes
├── studies/                Successive Millet snapshots and study examples
├── queries/                Saved SPARQL retrieval and checking queries
├── media/                  Figures and supporting screenshots
├── semrevtut.Rproj         RStudio project metadata
├── LICENSE                 GNU General Public License v3
└── README.md               Project overview and executable guide
```

### `code_books/`

- `BloomsTaxonomy.skos.ttl` — tutorial SKOS representation of the original and revised Bloom cognitive taxonomies.
- `DDI_ModeOfCollection_5.0.1.skos.ttl` — versioned DDI Mode of Collection vocabulary used to illustrate reuse of an external scheme.
- `DDI_TypeOfInstrument_1.1.2.skos.ttl` — versioned DDI Type of Instrument vocabulary; `queries/ddi-instrument-path.rq` traverses its questionnaire hierarchy.
- `research_methods.skos.ttl` — local research-methods scheme used in the provenance and cross-study examples.
- `tech_features.skos.ttl` — local technology-feature scheme used for classification and broader-concept retrieval.
- `education_levels.trig` — supplementary education-level concept scheme using ERIC terminology.

The companion query [`queries/list-bloom-concepts.rq`](queries/list-bloom-concepts.rq) lists the revised Bloom cognitive-process concepts.

### `studies/`

- `Millet2013_extracted.trig` — initial graph of the publication, study, learning activities, features, data, and claims.
- `Millet2013_annotations.trig` — self-contained next-stage snapshot adding passage-anchored Web Annotations.
- `Millet2013_provenance.trig` — self-contained final snapshot adding reviewers, activities, versions, statuses, disagreement, and adjudication.
- `Burgin2020_provenance.trig`, `Hoest2013_provenance.trig`, and `Lan2025_provenance.trig` — compact provenance snapshots that join Millet in the manuscript’s four-publication retrieval.
- `Hudson-Smith2023_provenance.trig` — an additional compatible provenance example retained as supplementary project data; it is not part of the manuscript’s four-publication query.

The three Millet files are cumulative conceptual stages, but each is an independently inspectable snapshot. Do not load all three together as though they were disjoint records.

### `queries/`

The saved `.rq` files are executable documentation. They demonstrate vocabulary traversal, annotation retrieval, claim retrieval, disagreement detection, selection of accepted records, provenance checks, cross-study joins, and hierarchical technology retrieval. Together with the exact RDF inputs and expected results above, they form the computationally reproducible part of the tutorial.

### `media/`

This directory contains the tutorial’s conceptual overview, annotation-model figure, and supporting diagrams or ARQ output screenshots used during manuscript development.

## Representation standards

The examples combine:

- **RDF** as the graph data model, serialized as Turtle (`.ttl`) and TriG (`.trig`);
- **SKOS** for concepts, concept schemes, labels, definitions, and semantic relations;
- **Web Annotation** for identified coding acts with bodies and passage-level targets;
- **PROV-O** for agents, activities, generation, attribution, and derivation;
- **SPARQL** for retrieval, checking, joins, and table-shaped views; and
- a small use of the **Micropublications ontology** to distinguish extracted data from reviewer-formulated claims and state support relations.

The raw RDF is shown for transparency and learning. A production coding form, qualitative-analysis application, review platform, or AI-assisted interface could generate and consume the same structures without requiring reviewers to write triples manually.

## External namespaces

The tutorial uses the following external vocabulary namespaces in addition to the project-owned namespaces under `http://www.learn-graph.net/`. Prefixes are abbreviations used in the example RDF and SPARQL files; they are not part of the underlying identifiers.

| Prefix | Namespace | Use in the tutorial |
|----|----|----|
| `bibo:` | `http://purl.org/ontology/bibo/` | Bibliographic resources and human-readable page locators. |
| `dct:` / `dcterms:` | `http://purl.org/dc/terms/` | Titles, sources, identifiers, dates, creators, descriptions, and other dataset or vocabulary metadata. |
| `dctype:` | `http://purl.org/dc/dcmitype/` | DCMI resource types, including the dataset type used for vocabulary metadata. |
| `foaf:` | `http://xmlns.com/foaf/0.1/` | Descriptions of people and other agents in supplementary project data. |
| `moc:` | `http://rdf-vocabulary.ddialliance.org/cv/ModeOfCollection/5.0.1/` | Version 5.0.1 of the DDI Mode of Collection controlled vocabulary. |
| `toi:` | `http://rdf-vocabulary.ddialliance.org/cv/TypeOfInstrument/1.1.2/` | Version 1.1.2 of the DDI Type of Instrument controlled vocabulary. |
| `mp:` | `http://purl.org/mp/` | Micropublications classes and relations for extracted data, claims, and evidential support. |
| `oa:` | `http://www.w3.org/ns/oa#` | Web Annotations, motivations, targets, sources, and text-quote selectors. |
| `prov:` | `http://www.w3.org/ns/prov#` | PROV-O agents, entities, activities, attribution, generation, use, and derivation. |
| `rdf:` | `http://www.w3.org/1999/02/22-rdf-syntax-ns#` | Core RDF terms, including types. |
| `rdfs:` | `http://www.w3.org/2000/01/rdf-schema#` | Human-readable labels and basic schema terms. |
| `skos:` | `http://www.w3.org/2004/02/skos/core#` | Concept schemes, concepts, labels, definitions, notes, hierarchies, and mappings. |
| `owl:` | `http://www.w3.org/2002/07/owl#` | Vocabulary version information. |
| `xsd:` | `http://www.w3.org/2001/XMLSchema#` | Datatypes for dates, date-times, years, and other literal values. |
| `xml:` | `http://www.w3.org/XML/1998/namespace` | The XML namespace declared by supplementary vocabulary data. |

## Reuse and citation

The project is licensed under the [GNU General Public License v3](LICENSE). Consult [`references.bib`](references.bib) and the manuscript for the sources cited by the tutorial and its running examples. Source publications themselves are not included; the study graphs retain bibliographic descriptions, passage selectors, and permitted quotations needed to connect review-generated data to the evidence.
