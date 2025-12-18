   # Metagenomic Viral Detection Tool Assessment Report
## Presentation for CDC Expert Review

**Author**: Sihua Peng  
**Assessment Environment**: UGA Sapelo2 (HPC Cluster)  
**Date**: 2025-12-15

---

## Slide 1: Title Page

# Metagenomic Viral Detection Tool Assessment Report

**Presentation to CDC National Wastewater Surveillance System (NWSS) Technical Review Committee**

- **Assessment Objects**: 10 CDC-listed software + 3 self-developed workflows
- **Assessment Environment**: UGA Sapelo2 HPC Cluster (no sudo restrictions)
- **Core Objectives**: Engineering-implementable + Scientifically-interpretable

**Sihua Peng**  
December 15, 2025

---

## Slide 2: Assessment Background and Objectives

### CDC Assessment Task

**Assessment List**: 10 software/platforms

**Assessment Focus**:
- ✅ **Deployability and Reproducibility** (cluster deployment without sudo)
- ✅ **Core Algorithms and Dependency Ecosystem**
- ✅ **Short/Long Read Compatibility**
- ✅ **Result Interpretation and False Positive Risk**

**Assessment Principles**:
- Default output vs. Consensus/Threshold output
- Avoid contradictory conclusions like "both most sensitive and most specific"

---

## Slide 3: Assessment Results Overview

### Assessment Completion Status

| Category | Count | Status |
|:---|:---:|:---|
| **CDC List - Runnable** | 5 | ✅ In-depth Assessment |
| **CDC List - Not Runnable** | 5 | ❌ Reasons Documented |
| **Author-Developed Workflows** | 3 | ✅ In-depth Assessment |
| **Total In-depth Assessment** | **8** | ✅ |

**Key Finding**: Deployability is a critical threshold for tool selection

---

## Slide 4: 8 Tools for In-Depth Assessment

### Within CDC List (5 items)

1. **nf-core/taxprofiler** - Multi-classifier parallel workflow
2. **GOTTCHA2** - High-specificity signature classification
3. **sourmash** - MinHash rapid retrieval
4. **CLARK** - Discriminative k-mer classification
5. **nf-core/mag** - Metagenomic assembly and binning

### Author-Developed (3 items)

6. **rvdb-viral-metagenome-nf** - Viral discovery workflow
7. **MLMVD-nf** - Multi-tool consensus screening
8. **KrakenMetaReads-nf** - Assembly before classification

---

## Slide 5: Test Datasets

### Data Used for Assessment

**Short Read Data (Illumina)**:
- `llnl_66ce4dde_R1.fastq.gz` (670.52 MB)
- `llnl_66ce4dde_R2.fastq.gz` (693.43 MB)

**Long Read Data (ONT/PacBio-like)**:
- `llnl_66d1047e.fastq.gz` (263.93 MB)

> **Note**: This is a simulated dataset from CDC

---

## Slide 6: Software Unable to be Assessed

### 5 Items from CDC List Failed Deployment

| Software | Main Obstacle |
|:---|:---|
| **CZID (IDseq)** | Cloud service oriented; local miniwdl insufficient cluster adaptation |
| **SURPI+** | Container requires sudo; conflicts with Apptainer security policy |
| **DHO Lab** | Container sudo dependency; complex source dependency chain |
| **NAO MGS** | Container sudo dependency; complex source dependency chain |
| **TaxTriage** | Container sudo dependency; complex source dependency chain |

**Common Reason**: Containerized deployment requires sudo permissions, incompatible with HPC principle of least privilege

---

## Slide 7: Tool Classification and Strategies

### Four Analysis Strategies

1. **Direct Read Classification**
   - taxprofiler (Kraken2/Bracken)
   - CLARK, GOTTCHA2

2. **Contig Classification After Assembly**
   - KrakenMetaReads-nf
   - nf-core/mag

3. **Homology Evidence Chain**
   - rvdb-viral (DIAMOND/RVDB)

4. **Viral Feature/Consensus Screening**
   - MLMVD-nf (multi-tool consensus)

---

## Slide 8: nf-core/taxprofiler

### Multi-Classifier Parallel Workflow

**Positioning**: Broad screening + Rapid overview

**Core Features**:
- Nextflow + nf-core best practices
- Supports multiple classifiers (Kraken2, MetaPhlAn, Kaiju, etc.)
- Automatic adaptation to short/long reads

**Database:** Pre-built Kraken2 viral database

**Read Compatibility**: ✅ Fully Compatible

**Usage Recommendations**: 
- Broad screening → Priority use
- High-confidence list → Need to apply consensus/threshold strategies

---

## Slide 9: GOTTCHA2

### High-Specificity Signature Classification

**Positioning**: High-specificity classification based on unique signature fragments

**Core Concept**: Only reports when hitting species-specific regions

**Read Compatibility**: ✅ Compatible (Minimap2 friendly to long reads)

**Advantages**: Extremely low false positive rate

**Limitations**: May miss extremely low abundance, database-missing targets

**Database**: RefSeq Bacteria, Archaea and Viruses with the gottcha_db.species.fna.mmi file of 100.35 Gb.

**Usage Recommendation**: Use as "secondary confirmation" tool

---

## Slide 10: sourmash

### MinHash Rapid Retrieval

**Positioning**: Rapid similarity/containment judgment

**Advantages**: 
- Extremely fast, resource-light
- Suitable for large-scale retrieval

**Read Compatibility**: ✅ Compatible (but error rates affect k-mer matching)

**Database:** Sourmash NCBI Viruses database (pre-built)

**Usage Recommendations**: 
- Coverage/consistency validation
- Long reads need error correction or use as "coarse validation"

---

## Slide 11: CLARK

### Discriminative k-mer Classification

**Positioning**: Uses only k-mers that uniquely distinguish species

**Variants**: CLARK (standard), CLARK-S (spaced seeds), CLARK-l (lightweight)

**Database:** NCBI RefSeq Virus Database

**Advantages**: Fast speed, good specificity

**Risks**: 
- Strong dependency on database coverage
- Long read errors affect exact k-mer matching

**Usage Recommendation**: Recommend CLARK-S for long reads, requires sufficient memory

---

## Slide 12: nf-core/mag

### Metagenomic Assembly and Binning

**Positioning**: Genome reconstruction (not targeting detection)

**Core Value**: Produces contigs, bins, MAGs usable for evolutionary/functional analysis

**Database:** Kraken2 Viral Database

**Read Compatibility**: 
- ✅ Short reads fully compatible
- ✅ Hybrid assembly friendly
- ⚠️ Pure long read assembly strategy not optimal

**Applicable Scenarios**: Analyses requiring high-quality genome reconstruction

---

## Slide 13: rvdb-viral-metagenome-nf

### Viral Discovery-Oriented Workflow

**Positioning**: Assembly + RVDB homology search + viral feature validation

**Core Objective**: Improve discovery capability for environmental/distant viruses

**Database:** RVDB DNA (Virual classification)+RVDB protein (Protein alignment)

**Advantages**: 
- RVDB database optimized for viruses
- Protein-level search discovers distant sequences
- Particularly suitable for large viruses like NCLDV

**Read Compatibility**: ✅ Compatible (long reads more beneficial for large virus reconstruction)

---

## Slide 14: MLMVD-nf & KrakenMetaReads-nf

### MLMVD-nf: Multi-Tool Consensus Screening

**Positioning**: Strict filter (achieving higher specificity at the expense of sensitivity)

**Database:** 
  
- DeepVirFinder does not rely on any viral reference sequence database.  
- VirSorter2: VirSort2 database with 12 Gb. It mainly consists of viral sequences + a small amount of non-viral training data.  
- viralFlye: Pfam database  

**Strategy**: VirSorter2 + DeepVirFinder + viralFlye voting

**Output**: Smaller but more reliable candidate set

### KrakenMetaReads-nf: Assembly Before Classification

**Positioning**: Uses long contigs to reduce classification ambiguity

**Database:** Kraken2 viral reference database

**Advantage**: Particularly suitable for giant viruses/large genome fragment positioning

**Risk**: Low-abundance taxa may be lost during assembly stage

---

## Slide 15: Empirical Results - taxprofiler

### Key Observations from Short Read Sample

**Advantages**:
- Broad-spectrum detection, forms "long-tail distribution"
- Can capture large numbers of low-abundance signals

**Risk Points**:
- Taxa with "reads < 10" heavily affected by database bias and contamination
- Must be combined with thresholds and coverage evidence

**Conclusion**: Suitable for initial screening, requires subsequent validation

---

## Slide 16: Empirical Results - Assembly Strategy Comparison

### MEGAHIT vs SPAdes

**SPAdes Advantages**:
- Produces more continuous viral contigs
- Beneficial for subsequent evidence chain construction

**MEGAHIT Advantages**:
- More "comprehensive", retains more low-abundance fragments

**Consensus Strategy**: Contig sets overlapping between the two assemblers are more robust

### Long Read Assembly Advantages

- Flye/viralFlye can produce contigs from tens of kb to hundreds of kb
- Provides strong evidence chains for large viruses like NCLDV

---

## Slide 17: Empirical Results - Discovery-Oriented Tools

### rvdb-viral Key Observations

**Consensus Set**: Dual-track (assembly/features/homology) overlap more reliable

**NCLDV Clues**: 
- Large numbers of Mimiviridae/Phycodnaviridae-related hits in long read samples
- Combined with long contigs forms strong evidence chain

### MLMVD-nf Key Observations

**3-tool consensus**: Strict consensus converges to small number of high-confidence contigs

**Recommendation**: Treat as "confirmation candidate set generator", not "broad-spectrum detector"

---

## Slide 18: Computational Resources and Engineering Costs

### Resource Consumption Categories

**CPU/Time Intensive**:
- nf-core/mag
- rvdb-viral
- MLMVD-nf

**Memory Intensive**:
- Kraken2/CLARK (database indices)

**Lightweight**:
- sourmash (extremely small sketch)

### Engineering Maturity

**High**: nf-core series, Nextflow workflows (batch processing, logging, checkpoint resumption)

**Medium**: Standalone tools (suitable for point-to-point validation)

---

## Slide 19: Database is the "Invisible Deciding Factor"

### Impact of Database Selection

**RefSeq/Standard Libraries**:
- ❌ Often underestimate environmental virus and phage diversity

**RVDB**:
- ✅ More friendly to viruses (especially environmental viruses and distant sequences)
- ⚠️ Need to guard against database annotation bias

**Conclusion**: Database selection directly affects discovery capability

---

## Slide 20: False Positive Control Strategies

### Minimum Essential Strategy (Strongly Recommended)

1. **Host Removal**
   - Short reads: Bowtie2
   - Long reads: Minimap2

2. **Threshold Filtering**
   - Set minimum read count/relative abundance thresholds

### Multi-Evidence Chain Consensus

**Strategy A**: Multi-tool consensus (≥2 tools support)

**Strategy B**: Assembly evidence priority
- Require medium-to-long contigs (>5-10kb)
- Viral hallmark genes
- DIAMOND/RVDB or VirSorter2/CheckV support

---

## Slide 21: Decision Matrix (Scores 1-5)

| Dimension | taxprofiler | rvdb-viral | MLMVD-nf | nf-core/mag | GOTTCHA2 | CLARK | sourmash | KrakenMetaReads |
|:---|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|
| **Known Virus Sensitivity** | 5 | 4 | 2 | 2* | 3 | 3 | 3 | 3 |
| **Distant Virus Discovery** | 3 | 5 | 3 | 4 | 1 | 1 | 2 | 3 |
| **Specificity** | 3 | 3 | 5 | 4* | 5 | 4 | 4 | 3 |
| **Result Reliability** | 4 | 4 | 4 | 5 | 4 | 3 | 4 | 4 |
| **Analysis Speed** | 3 | 1 | 2 | 1 | 4 | 5 | 5 | 3 |
| **Memory Efficiency** | 2 | 3 | 3 | 2 | 4 | 2 | 5 | 2 |
| **Genome Reconstruction** | 1 | 5 | 2 | 5 | 1 | 1 | 1 | 4 |
| **Engineering Maturity** | 5 | 5 | 4 | 5 | 3 | 3 | 4 | 5 |

\* nf-core/mag scores are **indirect assessments** via MAG reconstruction and classification, not direct read-level detection. High specificity comes from MAG QC evidence chain (CheckM/GUNC/coverage), not read-level classification.

---

## Slide 22: Recommended Solution One

### High-Confidence Consensus and Benchmark Analysis

**Tool Combination**: 
- taxprofiler (primary screening)
- GOTTCHA2 / sourmash / assembly evidence (key target confirmation)

**Workflow**:
1. taxprofiler broad-spectrum detection + rapid overview
2. Host removal + thresholds + coverage validation
3. GOTTCHA2 signature coverage confirmation

**Applicable Scenarios**: 
- Wastewater environmental virus monitoring
- Pre-clinical screening projects

---

## Slide 23: Recommended Solutions Two & Three

### Solution Two: Emerging/Outbreak Pathogen Discovery

**Tool Combination**: 
rvdb-viral (discovery) → MLMVD-nf (strict screening) → nf-core/mag (deep reconstruction)

**Applicable**: Environmental unknown virus surveys, outbreak tracing

### Solution Three: Wastewater Virus Monitoring

**Tool Combination**: 
taxprofiler (batch screening) → GOTTCHA2 (confirmation) → KrakenMetaReads-nf/rvdb-viral (assembly validation)

**Key Points**: Multi-evidence chains + coverage thresholds + secondary confirmation

**Output**: Distinguish "screen-positive" from "confirm-positive"

---

## Slide 24: Final Recommendations and Conclusions

### Core Recommendations

1. **Deployment Reproducibility is a Prerequisite**
   - Tools that cannot run stably should not enter public health decision chains

2. **Never Use Single Read Count as Existence Conclusion**
   - Must use coverage/assembly/multi-tool consistency

3. **Establish Evidence Tier Labels**
   - Screen-positive / Supportive / Confirmed

### Key Conclusions

- ✅ No single tool can meet all requirements
- ✅ Need layered strategy: Rapid screening → Confirmation → Deep analysis
- ✅ Deployability is a critical threshold for tool selection
- ✅ Multi-evidence chain consensus is key to reducing false positives

---

## Slide 25: Acknowledgments and Contact Information

### Acknowledgments

**CDC National Wastewater Surveillance System (NWSS) Technical Review Committee**

### Contact Information

**Sihua Peng**  
UGA Sapelo2 HPC Cluster

### Report Access

Full Assessment Report: `metagenome_viral_tool_assessment_rev5.md`

---

**End of Presentation. Thank You!**









