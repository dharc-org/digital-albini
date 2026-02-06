# Digital Albini

[![License](https://img.shields.io/badge/license-CC--BY--4.0-blue.svg)](LICENSE)
[![RDF](https://img.shields.io/badge/format-RDF%2FTTL-orange.svg)](letters_dataset/data/output/letters_kg.ttl)

The Giuseppe Albini Archive digitization project transforms hierarchical archival metadata into a semantic knowledge graph using the **Records in Contexts (RiC-O) ontology**. The conversion process is implemented in Python and produces RDF/Turtle output suitable for SPARQL querying and linked data integration.


### Features

- **Semantic Modeling**: Uses RiC-O ontology for archival description
- **External Enrichment**: Integrates GeoNames data for geographic entities
- **Authority Control**: Links to VIAF for persons and corporate bodies
- **Hierarchical Preservation**: Maintains archive's original record structure

## Table of Contents

1. [Archive Structure](#archive-structure)
2. [Technical Architecture](#technical-architecture)
3. [Dependencies and Configuration](#dependencies-and-configuration)
4. [Ontologies and Namespaces](#ontologies-and-namespaces)
5. [Output Specifications](#output-specifications)

---

## Archive Structure

The Albini collection follows a hierarchical organization mapped to RiC-O classes:

```
Serie (Series)
└── Sottoserie (Sub-series)
    └── Busta (Folder/Box)
        └── Fascicolo (File)
            └── Documento (Document)
                └── Immagine (Image)
```

### Hierarchical Mapping

| Level | Italian Term | RiC-O Class | Count |
|-------|-------------|-------------|-------|
| 1 | Serie | `rico:RecordSet` | 4 |
| 2 | Sottoserie | `rico:RecordSet` | 7 |
| 3 | Busta | `rico:RecordSet` | 36 |
| 4 | Fascicolo | `rico:RecordSet` | 581 |
| 5 | Documento | `rico:Record` | 1,852 |
| 6 | Immagine | `rico:RecordResource` | 6,184 |

---

## Technical Architecture

### Input Files

The conversion process requires two Excel workbooks:

1. **`mapping1.xlsx`** - Defines column-to-predicate mappings for each archival level
   - Each sheet corresponds to an archival level (Serie, Sottoserie, Fascicolo, Documento)
   - Columns: `Subject`, `Predicate`, `Object`, `Column Subject`, `Column Object`

2. **`instances.xlsx`** - Contains the actual archival metadata
   - Sheet names must match those in mapping file (case-insensitive)
   - Each row represents an archival entity with its descriptive metadata

### Output File

- **`letters_kg.ttl`** - RDF knowledge graph in Turtle format
- Base namespace: `http://www.w3.org/ns/DigitalAlbini#`
- Fully self-contained with all entity definitions and relationships

### Script Location and Execution

```bash
# File structure
digital-albini/
├── letters_dataset/
│   ├── src/
│   │   └── script.py          # Main conversion script
│   ├── data/
│   │   ├── input/
│   │   │   ├── mapping1.xlsx
│   │   │   └── instances.xlsx
│   │   └── output/
│   │       └── letters_kg.ttl
│   └── docs/
│       ├── busta_uml.jpg
│       ├── serie_uml.jpg
│       └── sottoserie_uml.jpg

# Execute conversion
cd letters_dataset/src
python script.py
```

---

## Dependencies and Configuration

### Required Python Libraries

```python
import pandas as pd        # Excel file processing
import re                  # Regular expressions for parsing
import os                  # File path handling
import requests            # GeoNames API calls
from rdflib import Graph, Namespace, URIRef, Literal
from rdflib.namespace import RDF, RDFS, XSD, OWL
from datetime import datetime
```

**Optional but Recommended:**
```bash
pip install geonamescache  # Local GeoNames database cache
```

### Configuration Constants

```python
# File paths (relative to project root)
BASE_DIR = os.path.dirname(os.path.abspath(__file__))
PROJECT_ROOT = os.path.dirname(BASE_DIR)
mapping_path = os.path.join(PROJECT_ROOT, "data", "input", "mapping1.xlsx")
instances_path = os.path.join(PROJECT_ROOT, "data", "input", "instances.xlsx")
output_path = os.path.join(PROJECT_ROOT, "data", "output", "letters_kg.ttl")

# Namespace configuration
BASE_NS = "http://www.w3.org/ns/DigitalAlbini#"

# External services
GEONAMES_USERNAME = "your_username_here"  # Required for API access
```

### GeoNames Integration

The script supports two modes for geographic enrichment:

1. **Local Cache (geonamescache)** - Fast, offline lookups
2. **API Fallback** - Online queries when local data unavailable

**Setup GeoNames Account:**
1. Register at https://www.geonames.org/login
2. Enable free web services in account settings
3. Replace `GEONAMES_USERNAME` in script configuration

---

## Ontologies and Namespaces

### Primary Ontology

**Records in Contexts Ontology (RiC-O) 0.2**
- URL: https://www.ica.org/standards/RiC/ontology
- Developed by: International Council on Archives (ICA)
- Purpose: Archival description and relationships

### Supporting Ontologies

| Prefix | Namespace | Purpose |
|--------|-----------|---------|
| `rico` | https://www.ica.org/standards/RiC/ontology# | Core archival classes and properties |
| `wgs84` | http://www.w3.org/2003/01/geo/wgs84_pos# | Geographic coordinates |
| `gn` | http://www.geonames.org/ontology# | GeoNames feature classification |
| `owl` | http://www.w3.org/2002/07/owl# | Ontology alignment (sameAs) |
| `xsd` | http://www.w3.org/2001/XMLSchema# | Datatype definitions |
| `rdfs` | http://www.w3.org/2000/01/rdf-schema# | Labels and schema |

### Custom Namespaces (Digital Albini)

All local entities use the base namespace: `http://www.w3.org/ns/DigitalAlbini#`

| Namespace | Usage | Example URI |
|-----------|-------|-------------|
| `record/` | Individual documents (rico:Record) | `DigitalAlbini#record/S1_SS1_B1_F1_D1` |
| `recordset/` | Aggregations (rico:RecordSet) | `DigitalAlbini#recordset/S1_SS1_B1` |
| `person/` | Individual agents | `DigitalAlbini#person/Giovanni_Gentile` |
| `corporateBody/` | Institutional agents | `DigitalAlbini#corporateBody/University_of_Bologna` |
| `place/` | Geographic locations | `DigitalAlbini#place/Roma` |
| `physloc/` | Physical locations | `DigitalAlbini#physloc/Bologna` |
| `date/` | Temporal entities | `DigitalAlbini#date/1920` |
| `title/` | Document titles | `DigitalAlbini#title/Corrispondenza_1920` |
| `internalIdentifier/` | Archive reference codes | `DigitalAlbini#internalIdentifier/S1_SS1_B1_F1` |
| `storageid/` | Physical storage identifiers | `DigitalAlbini#storageid/box12` |
| `inst/` | Physical instantiations | `DigitalAlbini#inst/S1_SS1_B1_F1_D1` |
| `type/` | Identifier types | `DigitalAlbini#type/InternalIdentifier` |
| `temp/` | Internal processing (removed in final output) | `temp:intermediateSender` |

---

## Output Specifications

### File Format

- **Format:** Turtle (RDF 1.1)
- **Encoding:** UTF-8
- **File Extension:** `.ttl`
- **MIME Type:** `text/turtle`

## References

### Standards and Ontologies

- **RiC-O 0.2**: https://www.ica.org/standards/RiC/ontology
- **RDF 1.1**: https://www.w3.org/TR/rdf11-primer/
- **Turtle**: https://www.w3.org/TR/turtle/
- **SKOS**: https://www.w3.org/TR/skos-reference/
- **Dublin Core**: https://www.dublincore.org/specifications/dublin-core/dcmi-terms/

### External Services

- **GeoNames**: https://www.geonames.org/
- **VIAF**: https://viaf.org/
- **geonamescache**: https://pypi.org/project/geonamescache/

### Related Documentation

- UML Data Model Diagrams: See `docs/` folder
  - `busta_uml.jpg` - Folder-level model
  - `serie_uml.jpg` - Series-level model
  - `sottoserie_uml.jpg` - Sub-series level model

---

**Last Updated:** February 2025  
**Version:** 1.0  
**Maintainer:** Digital Albini Project Team  
**Institution:** University of Bologna - FICLIT
