# SDRF Templates Guide

A comprehensive guide for template selection, combination rules, key column guidance, and format conventions. It complements the per-template YAML definitions and auto-generated documentation.

## Template Architecture

### Layers

Templates are organized in four layers:

```
base                         Infrastructure (identifiers, data files, versioning)
  sample-metadata            Shared sample biology (organism, tissue, disease)
    technology templates     MS or affinity proteomics columns
    organism templates       Species-specific metadata
    experiment templates     Technique-specific columns
    clinical templates       Clinical/oncology metadata
```

### Inheritance

Every template inherits all columns from its parent chain. Child templates can override parent column properties (e.g., changing a column from `optional` to `required`).

```
base
 └── sample-metadata
      ├── ms-proteomics (technology)
      │    ├── dia-acquisition (experiment)
      │    ├── crosslinking (experiment)
      │    ├── immunopeptidomics (experiment)
      │    ├── single-cell (experiment)
      │    └── metaproteomics (experiment)
      ├── affinity-proteomics (technology)
      │    ├── olink (experiment)
      │    └── somascan (experiment)
      ├── human (sample/organism)
      ├── vertebrates (sample/organism)
      ├── invertebrates (sample/organism)
      ├── plants (sample/organism)
      ├── cell-lines (experiment)
      ├── clinical-metadata (sample/clinical)
      │    └── oncology-metadata (sample/clinical)
      └── (columns: organism, organism part, cell type, disease, biological replicate...)
```

### Combination Rule

An SDRF file is built by **combining** templates:

1. **ONE technology template** (required): `ms-proteomics` or `affinity-proteomics`
2. **ONE organism template** (recommended): `human`, `vertebrates`, `invertebrates`, or `plants`
3. **ZERO or more experiment templates**: `dia-acquisition`, `single-cell`, `crosslinking`, `immunopeptidomics`, `metaproteomics`, `olink`, `somascan`
4. **ZERO or more clinical templates**: `clinical-metadata`, `oncology-metadata`
5. **ZERO or one cell-lines template**: when samples are cultured cell lines

When templates are combined, columns from all selected templates are merged. If the same column appears in multiple templates, the most specific definition wins (experiment > technology > sample-metadata).

### Mutual Exclusivity

- **Technology**: `ms-proteomics` and `affinity-proteomics` cannot be combined
- **Organism**: only one of `human`, `vertebrates`, `invertebrates`, `plants` per study (multi-species studies use the most specific applicable template)
- **Platform**: `olink` and `somascan` cannot be combined
- **Affinity experiments**: `olink`/`somascan` require `affinity-proteomics` (not `ms-proteomics`)

---

## How to Choose Templates

### By Study Type

| Study type | Templates to combine |
|------------|---------------------|
| Human clinical DDA proteomics | `human` + `ms-proteomics` + `clinical-metadata` |
| Human cancer proteomics | `human` + `ms-proteomics` + `oncology-metadata` |
| Mouse DDA proteomics | `vertebrates` + `ms-proteomics` |
| Plant stress proteomics | `plants` + `ms-proteomics` |
| Human cell line drug screen | `cell-lines` + `human` + `ms-proteomics` + `clinical-metadata` |
| Human XL-MS structural study | `crosslinking` + `human` |
| In-vitro XL-MS (purified proteins) | `crosslinking` |
| Human immunopeptidomics / HLA ligandomics | `immunopeptidomics` + `human` |
| Mouse immunopeptidomics / H-2 ligandomics | `immunopeptidomics` + `vertebrates` |
| Human DIA proteomics | `dia-acquisition` + `human` |
| Mouse DIA proteomics | `dia-acquisition` + `vertebrates` |
| Human single-cell proteomics | `single-cell` + `human` |
| Human gut metaproteomics | `metaproteomics` + `human` |
| Environmental metaproteomics (soil, ocean) | `metaproteomics` |
| Human Olink plasma study | `olink` + `human` |
| Human SomaScan serum study | `somascan` + `human` |
| Drosophila phosphoproteomics | `invertebrates` + `ms-proteomics` |
| Zebrafish developmental proteomics | `vertebrates` + `ms-proteomics` |
| Arabidopsis drought response proteomics | `plants` + `ms-proteomics` |
| Mouse cancer model proteomics | `vertebrates` + `ms-proteomics` + `oncology-metadata` |

### By Organism

| Organism | Template |
|----------|----------|
| Homo sapiens | `human` |
| Mouse, rat, zebrafish, other vertebrates | `vertebrates` |
| Drosophila, C. elegans, insects | `invertebrates` |
| Arabidopsis, rice, maize, crops | `plants` |
| Cultured cell lines (any origin) | `cell-lines` + organism template matching cell origin |

### By Technology

| Technology | Template |
|------------|----------|
| Any mass spectrometry proteomics | `ms-proteomics` |
| DDA (data-dependent acquisition) | `ms-proteomics` (default) |
| DIA / SWATH / diaPASEF | `dia-acquisition` |
| TMT / iTRAQ / SILAC | `ms-proteomics` (labeling defined by `comment[label]`) |
| Label-free quantification | `ms-proteomics` (label = "label free sample") |
| Olink PEA | `olink` (inherits `affinity-proteomics`) |
| SomaScan aptamer | `somascan` (inherits `affinity-proteomics`) |

---

## Key Columns Explained

### Sample Identity

- **`source name`** (base): Unique sample identifier. One per biological sample. Required for every row.
- **`assay name`** (base): Unique data acquisition run identifier. Links sample to instrument run.
- **`characteristics[individual]`** (human, single-cell): Links multiple samples to the same donor/patient. Critical for matched tumor/normal designs, longitudinal studies, and single-cell experiments where multiple cells come from one donor.
- **`characteristics[biological replicate]`** (sample-metadata): Integer starting from 1. Use `pooled` for pooled reference samples.
- **`comment[sample type]`** (sample-metadata): Classifies the sample's role in multiplexed experiments. Values include: `single cell`, `reference`, `bridge`, `carrier`, `empty`, `quality control sample`, `negative control`, `positive control`, `bulk control`, `standard`, `calibrator`, `plate control`, `buffer control`.

### Disease and Clinical Context

- **`characteristics[disease]`** (sample-metadata): The disease state. Use `normal` (PATO:0000461) for healthy samples. Uses MONDO, EFO, or DOID ontologies.
- **`characteristics[disease staging]`** (oncology-metadata): General stage (I-IV). For cancer, use the specialized oncology-metadata columns.
- **`characteristics[tumor stage]`** (oncology-metadata): TNM notation (e.g., T2N1M0). Prefix: p = pathological, c = clinical.
- **Specialized staging systems** (oncology-metadata):
  - `characteristics[dukes stage]` — colorectal cancer (A, B, C, D)
  - `characteristics[ann arbor stage]` — lymphoma (I-IV with optional A/B/E/S suffix)
  - `characteristics[gleason score]` — prostate cancer (sum or components: 3+4, 4+3)
  - `characteristics[weiss grade]` — adrenal cortical carcinoma (low, high)

### Treatment Annotation

- **`characteristics[treatment]`** (clinical-metadata): What was applied (drug name, stimulus). Use `untreated` for controls.
- **`characteristics[compound]`** + **`characteristics[dose]`** + **`characteristics[exposure duration]`** (clinical-metadata): Detailed drug annotation. Use together for pharmacological studies.
- **`characteristics[treatment status]`** (clinical-metadata): When sampled relative to treatment: `pre-treatment`, `on treatment`, `post-treatment`, `treatment naive`.
- **`characteristics[treatment response]`** (clinical-metadata): Clinical response: `complete response`, `partial response`, `progressive disease`, `stable disease`.

### Organism and Demographics

- **`characteristics[organism]`** (sample-metadata): NCBI Taxonomy species name, lowercase (e.g., `homo sapiens`, `mus musculus`).
- **`characteristics[sex]`** (human, vertebrates): `male`, `female`, `intersex` (human) or `hermaphrodite` (vertebrates).
- **`characteristics[age]`** (human): Strict format Y>M>W>D (e.g., `45Y`, `6M`, `30Y6M`, `40Y-50Y`). Special values: `not available`, `anonymized`, `pooled`.
- **`characteristics[ancestry category]`** (human): HANCESTRO terms (e.g., `European`, `African`, `East Asian`, `Hispanic or Latin American`).
- **`characteristics[strain or breed]`** (vertebrates, invertebrates, plants): Strain/cultivar/ecotype. Required for invertebrates, recommended for vertebrates and plants.
- **`characteristics[developmental stage]`**: Ontology depends on organism:
  - Human: HsapDv or EFO (`adult`, `fetal stage`)
  - Vertebrates: EFO or UBERON (`adult`, `embryo`, `juvenile`)
  - Invertebrates: FBdv (Drosophila), WBls (C. elegans), EFO, UBERON
  - Plants: PO or EFO (`seedling stage`, `flowering stage`)

### Cell Line Metadata

- **`characteristics[cell line]`** (cell-lines): Cell line name from CLO, BTO, or EFO (e.g., `HeLa`, `HEK293`, `MCF7`).
- **`characteristics[cellosaurus accession]`** (cell-lines): Required. Format CVCL_#### (e.g., `CVCL_0030`).
- **`characteristics[culture medium]`** (cell-lines): NCIT terms (e.g., `DMEM`, `RPMI 1640`).
- **`comment[passage number]`** (cell-lines): Integer or range (e.g., `10`, `15-20`).

### MS-specific Columns

- **`comment[instrument]`** (ms-proteomics): Mass spectrometer model. PSI-MS ontology terms (e.g., `Q Exactive HF`, `Orbitrap Fusion Lumos`, `timsTOF Pro`).
- **`comment[label]`** (ms-proteomics): Labeling strategy. Required. Examples: `label free sample`, `TMT126`, `TMTpro126`, `iTRAQ114`, `SILAC light`, `SILAC heavy`.
- **`comment[cleavage agent details]`** (ms-proteomics): Protease for digestion. Use ontology format: `NT=Trypsin;AC=MS:1001251`.
- **`comment[proteomics data acquisition method]`** (ms-proteomics): Acquisition mode. Values: `Data-dependent acquisition`, `Data-independent acquisition`, `Parallel reaction monitoring`, `Selected reaction monitoring`.
- **`comment[fraction identifier]`** (ms-proteomics): Integer. Use `1` for non-fractionated experiments. Required.
- **`comment[modification parameters]`** (ms-proteomics): PTM search parameters. Unimod format: `NT=Oxidation;MT=Variable;TA=M;AC=Unimod:35`. Multiple cardinality.

### DIA-specific Columns

- **`comment[scan window lower limit]`** / **`comment[scan window upper limit]`** (dia-acquisition): m/z boundaries of the DIA scan window (e.g., `400` to `1200`).
- **`comment[isolation window width]`** (dia-acquisition): Width of isolation windows in m/z (e.g., `25`, `8`, `4`).
- **`comment[dia method]`** (dia-acquisition): Specific DIA variant (e.g., `SWATH-MS`, `diaPASEF`, `MSE`, `All ion fragmentation`).

### Crosslinking-specific Columns

- **`comment[cross-linker]`** (crosslinking): Required. Structured format with linker identity and properties: `NT=DSS;AC=XLMOD:02001` or extended format `NT=DSSO;AC=XLMOD:02010;CL=yes;TA=K,S,T,Y,nterm;MH=54.01;ML=85.98`.
- **`characteristics[enrichment process]`** (crosslinking): Required. Must be `enrichment of cross-linked peptides`.
- **`comment[dissociation method]`** (crosslinking): Required (overridden from ms-proteomics). Dissociation affects cleavable linker stub ions.

### Immunopeptidomics-specific Columns

- **`characteristics[mhc protein complex]`** (immunopeptidomics): Required. Type of MHC complex targeted (class I, class II, non-classical).
- **`characteristics[mhc typing]`** (immunopeptidomics): IPD-IMGT/HLA notation for human (e.g., `HLA-A*02:01`), H-2 notation for mouse (e.g., `H-2Kb`). Semicolons for multi-allele.
- **`characteristics[antibody enrichment]`** (immunopeptidomics): Antibody clone used for immunoprecipitation (e.g., `W6/32`, `BB7.2`).

### Single-cell-specific Columns

- **`characteristics[single cell isolation protocol]`** (single-cell): Required. Isolation method (e.g., `FACS`, `cellenONE`, `LCM`, `nanoPOTS`).
- **`characteristics[cell identifier]`** (single-cell): Required. Unique per-cell label. Special values: `carrier`, `reference`, `empty`.
- **`comment[cells per well]`** (single-cell): Number of cells. Use `1` for true single cells. Recommended for SCP.
- **`comment[carrier channel]`** / **`comment[reference channel]`** (single-cell): TMT channels for carrier and reference in multiplexed SCP (e.g., `TMT131C`, `TMTpro134N`).

### Metaproteomics-specific Columns

- **`characteristics[environmental sample type]`** (metaproteomics): Required. ENVO/EFO term (e.g., `soil`, `seawater`, `gut microbiome`).
- **`characteristics[geographic location]`** (metaproteomics): GAZ terms or coordinates (e.g., `Pacific Ocean`, `47.6062 N, 122.3321 W`).
- **`characteristics[host organism]`** (metaproteomics): For host-associated samples. NCBI Taxonomy (e.g., `Homo sapiens`).
- **`characteristics[mock community]`** (metaproteomics): Mock community standard used (e.g., `ZymoBIOMICS Microbial Community Standard`).
- **`comment[metagenome accession]`** (metaproteomics): Matched metagenome dataset accession (e.g., `MGYA00001234`, `SRP123456`).

### Affinity Proteomics Columns

- **`comment[platform]`** (affinity-proteomics): Required. Platform name (e.g., `Olink Explore HT`, `SomaScan Assay 11K`).
- **`comment[sample matrix]`** (affinity-proteomics): Biological input material (e.g., `serum`, `plasma`, `cerebrospinal fluid`).
- **`comment[quantification unit]`** (affinity-proteomics): `NPX` for Olink, `RFU` for SomaScan.

### Data Files

- **`comment[data file]`** (base): Raw data filename. Must match the filename in the data repository. Required.
- **`comment[technical replicate]`** (base): Integer starting from 1. Required.

---

## Format Conventions

### Ontology Terms
- Use **lowercase** for organism names: `homo sapiens`, not `Homo sapiens`
- Use ontology-controlled vocabulary wherever validators specify ontology validation

### CV Term Format (Controlled Vocabulary)
Used for enzymes, modifications, instruments, and other ontology-mapped values:
```
NT={name};AC={accession}
```
Examples:
- `NT=Trypsin;AC=MS:1001251`
- `NT=Oxidation;MT=Variable;TA=M;AC=Unimod:35`
- `NT=DSS;AC=XLMOD:02001`

### Numbers with Units
Format: `{value} {unit}` (space-separated)
- Mass tolerance: `10 ppm`, `0.02 Da`
- Collision energy: `30 NCE`, `27 eV`
- Tissue mass: `50 mg`, `1 g`
- Cell diameter: `15 um`
- Temperature: `25 °C`, `-80 °C`
- Time: `24 hour`, `5 day`, `30 minute`

### Age Format (Human)
Strict order Y > M > W > D:
- `45Y` (45 years)
- `6M` (6 months)
- `30Y6M` (30 years, 6 months)
- `40Y-50Y` (age range)
- Special: `not available`, `anonymized`, `pooled`

### Special Values
- `not available` — value exists but is unknown
- `not applicable` — column does not apply to this sample
- `normal` — healthy/control for disease columns (PATO:0000461)
- `untreated` — no treatment applied (for treatment columns)
- `pooled` — pooled sample (for biological replicate)

### Multiple Values
Some columns allow multiple values (cardinality: multiple):
- Use multiple columns with the same header name
- Example: `characteristics[organism part]` can appear twice for mixed-tissue samples

### Accession Formats
- BioSample: `SAMN12345678`, `SAMEA12345678`, `SAMD1234567`
- Cellosaurus: `CVCL_0030`, `CVCL_0004`
- Metagenome: `MGYA00001234` (ENA), `SRP123456` (SRA)

---

## Versioning

### Template Versions
Templates follow semantic versioning (`MAJOR.MINOR.PATCH`):
- **MAJOR**: Breaking changes (column removals, requirement escalations)
- **MINOR**: Additions (new optional/recommended columns)
- **PATCH**: Fixes (description updates, validator corrections)

### Recording Templates in SDRF
Use `comment[sdrf template]` to record which templates were used:
```
NT=ms-proteomics;VV=v1.1.0    NT=human;VV=v1.1.0
```
Multiple template columns supported for combined templates.

---

## Template Reference

Each template's complete column definitions are in its YAML file. Key files:

| Template | Path | Usable alone |
|----------|------|-------------|
| base | `base/1.1.0/base.yaml` | No |
| sample-metadata | `sample-metadata/1.0.0/sample-metadata.yaml` | No |
| ms-proteomics | `ms-proteomics/1.1.0/ms-proteomics.yaml` | Yes |
| affinity-proteomics | `affinity-proteomics/1.0.0/affinity-proteomics.yaml` | Yes |
| human | `human/1.1.0/human.yaml` | No |
| vertebrates | `vertebrates/1.1.0/vertebrates.yaml` | No |
| invertebrates | `invertebrates/1.1.0/invertebrates.yaml` | No |
| plants | `plants/1.1.0/plants.yaml` | No |
| cell-lines | `cell-lines/1.1.0/cell-lines.yaml` | No |
| clinical-metadata | `clinical-metadata/1.0.0/clinical-metadata.yaml` | No |
| oncology-metadata | `oncology-metadata/1.0.0/oncology-metadata.yaml` | No |
| dia-acquisition | `dia-acquisition/1.1.0/dia-acquisition.yaml` | No |
| crosslinking | `crosslinking/1.0.0/crosslinking.yaml` | No |
| immunopeptidomics | `immunopeptidomics/1.0.0/immunopeptidomics.yaml` | No |
| single-cell | `single-cell/1.0.0/single-cell.yaml` | No |
| metaproteomics | `metaproteomics/1.0.0-dev/metaproteomics.yaml` | No |
| olink | `olink/1.0.0/olink.yaml` | No |
| somascan | `somascan/1.0.0/somascan.yaml` | No |
