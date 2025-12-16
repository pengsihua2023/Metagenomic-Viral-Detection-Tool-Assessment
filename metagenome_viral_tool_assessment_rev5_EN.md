# Metagenomic Viral Detection Tool Assessment Report
**Author**: Sihua Peng  
**Assessment Environment**: UGA Sapelo2 (HPC Cluster)  
**Date**: 2025-12-15

> This report is a "engineering-implementable + scientifically-interpretable" tool assessment and decision document. To avoid "listing without evidence" or "over-extrapolation of conclusions", this report adopts the following unified framework:  
> 1) **Default Output**: Refers to the default/common parameters and standard outputs used by tools in this assessment;  
> 2) **Consensus/Threshold Output**: Refers to the result set after applying additional rules such as "multi-tool cross-validation, coverage thresholds, host removal, etc." to the default output;  
> 3) Rankings and scores explicitly correspond to either "default output" or "consensus/threshold output", avoiding contradictory conclusions such as "both most sensitive and most specific".

---

## Table of Contents
1. [Assessment Background and Scope](#1-assessment-background-and-scope)  
2. [Software Unable to be Assessed (CDC List)](#2-software-unable-to-be-assessed-cdc-list)  
3. [Introduction: Evolution and Challenges of Metagenomic Analysis Paradigms](#3-introduction-evolution-and-challenges-of-metagenomic-analysis-paradigms)  
4. [In-Depth Tool Assessment (8 Runnable Objects)](#4-in-depth-tool-assessment-8-runnable-objects)  
5. [Empirical Results Analysis (Two Samples)](#5-empirical-results-analysis-two-samples)  
6. [Comprehensive Comparison and Key Metrics Assessment](#6-comprehensive-comparison-and-key-metrics-assessment)  
7. [False Positive Control Strategies: The Lifeline of Viral Detection](#7-false-positive-control-strategies-the-lifeline-of-viral-detection)  
8. [Decision Matrix (Scores 1–5)](#8-decision-matrix-scores-15)  
9. [Recommended Solutions and Expert Advice](#9-recommended-solutions-and-expert-advice)  
10. [Appendix](#10-appendix)  

---

## 1. Assessment Background and Scope

### 1.1 CDC List and Assessment Object Framework
The CDC requested assessment of **10 software/platforms**. This assessment was conducted on **UGA Sapelo2 (HPC Cluster)**, focusing on:

- **Deployability and Reproducibility** (cluster deployment without sudo, container/dependency management, batch execution capability);  
- **Core Algorithms and Dependency Ecosystem** (software stack and key steps called within workflows);  
- **Short/Long Read Compatibility** (input types, preprocessing, alignment/classification logic adaptation);  
- **Result Interpretation and False Positive Risk** (output interpretability, cross-validation pathways, coverage evidence).

Among the 10 items on the CDC list:  
- **5 items successfully completed deployment and runtime testing**, entering "in-depth assessment and comparison";  
- **5 items failed to complete deployment due to cluster permission and dependency compatibility issues**, with only failure reasons recorded (not included in performance comparison).

Additionally, to meet the business needs of viral assembly and "multi-evidence chain validation", this report also assesses **3 independently developed Nextflow workflows** by the author (not part of the original CDC list).

Therefore, **the in-depth assessment objects total 8**:  
- **Within CDC List and Runnable (5 items)**:  
  (1) nf-core/taxprofiler (hereinafter referred to as **taxprofiler / MetaTaxProfiler**)  
        https://github.com/pengsihua2023/MetaTaxProfiler
    
  (2) GOTTCHA2  
      https://github.com/pengsihua2023/GOTTCHA2-based-pipeline
    
  (3) sourmash  
      https://github.com/pengsihua2023/sourmash-based-pipeline
    
  (4) CLARK  
      https://github.com/pengsihua2023/CLARK-based-pipeline
    
  (5) nf-core/mag  
      https://github.com/pengsihua2023/mag-based-pipeline
    
- **Author-Developed (3 items)**:  
  (6) rvdb-viral-metagenome-nf (hereinafter referred to as **rvdb-viral**)  
      https://github.com/pengsihua2023/rvdb-viral-metagenome-nf
    
  (7) MLMVD-nf  
      https://github.com/pengsihua2023/MLMVD-nf
    
  (8) KrakenMetaReads-nf  
      https://github.com/pengsihua2023/KrakenMetaReads-nf
    


### 1.2 Successfully Assessed Software and Datasets

#### 1.2.1 Successfully Deployed and Running Software (Within CDC List)
- nf-core/taxprofiler  
- GOTTCHA2  
- sourmash  
- CLARK  
- nf-core/mag  

#### 1.2.2 Author-Developed Nextflow Workflows (Additional Assessment)
- rvdb-viral-metagenome-nf (MEGAHIT/SPAdes/viralFlye/Prodigal/DIAMOND; Database: RVDB)  
- MLMVD-nf (MEGAHIT/SPAdes/viralFlye + VirSorter2 + DeepVirFinder, etc., "multi-tool consensus")  
- KrakenMetaReads-nf (MEGAHIT/SPAdes/viralFlye/Flye + Kraken2/Bracken; emphasizes "assembly before classification")  

#### 1.2.3 Test Datasets (This is a simulated dataset from the CDC.)
- **Short Reads (Illumina)**:  
  - `llnl_66ce4dde_R1.fastq.gz` (670.52 MB)  
  - `llnl_66ce4dde_R2.fastq.gz` (693.43 MB)  
- **Long Reads (ONT/PacBio-like)**:  
  - `llnl_66d1047e.fastq.gz` (263.93 MB)  

---

## 2. Software Unable to be Assessed (CDC List)

Among the other 5 software items on the CDC list, none were able to complete reproducible deployment in the UGA Sapelo2 environment (no sudo, Apptainer security restrictions), and therefore could not enter empirical comparison.

### 2.1 Software Unable to be Assessed and Reasons
1. **CZID (IDseq)**: Primarily cloud-based service; local miniwdl path insufficiently adapted for cluster environments; container/runtime dependencies difficult to satisfy without sudo.  
2. **SURPI+**: Containerization path requires sudo permissions, conflicting with cluster Apptainer security policies; source compilation path encountered unresolvable dependency/compatibility issues.  
3. **DHO Lab**: Similar issues (container sudo dependency, complex source dependency chain).  
4. **NAO MGS**: Similar issues (container sudo dependency, complex source dependency chain).  
5. **TaxTriage**: Similar issues (container sudo dependency, complex source dependency chain).

### 2.2 Summary of Failure Reasons (Common Issues)
- **Containerization Deployment Barriers**: Image building/running methods requiring sudo do not comply with HPC's principle of least privilege;  
- **Source Compilation Difficulties**: Underlying dependencies (compilers/libraries/system components) conflict with cluster module environments, cannot be resolved at reasonable cost.

> **Conclusion**: In actual public health or research production environments, **deployability itself is a critical threshold for tool selection**.

---

## 3. Introduction: Evolution and Challenges of Metagenomic Analysis Paradigms

Metagenomics is evolving from a paradigm of "single sequencing technology + single classifier" to "hybrid sequencing + multi-evidence chain integration". Short reads (Illumina) provide high accuracy but fragmentation; long reads (ONT/PacBio) provide stronger connectivity and structural resolution, but with more complex error patterns and coverage structures. This change requires workflow design to simultaneously consider:

- **Preprocessing Adaptation**: Short reads emphasize adapter/quality trimming; long reads emphasize adapter removal, length/quality joint filtering (and error correction when necessary);  
- **Alignment/Classification Strategy Adaptation**:  
  - k-mer classifiers (Kraken2/CLARK/sourmash) emphasize speed but are sensitive to error rates and database bias;  
  - Alignment-based methods (Minimap2/DIAMOND/MALT) emphasize evidence chains and can be used for nucleic acid or protein-level validation;  
  - Viral mining (VirSorter2/DeepVirFinder/viralFlye, etc.) emphasizes "viral features" rather than homology, can discover distant sequences but require strict false positive control.  
- **Interpretation Framework**: The key to viral detection is not just a "detection list", but "coverage, distribution uniformity, genomic structural evidence, multi-tool consistency".

Therefore, the core objective of this report is not simply "which is stronger", but:  
1) Explain each tool's **core software ecosystem and evidence chain**;  
2) Compare differences between "default output" and "consensus/threshold output" on short/long read samples;  
3) Provide implementable **strategic combinations** for scenarios such as wastewater monitoring, environmental surveys, and outbreak tracing.

---

## 4. In-Depth Tool Assessment (8 Runnable Objects)

> **Chapter Focus**: For each tool, provide (1) core positioning, (2) core dependency ecosystem, (3) short/long read compatibility and risk points, (4) recommended usage.

### 4.1 nf-core/taxprofiler (MetaTaxProfiler)
**Positioning**: Multi-classifier parallel metagenomic classification and profiling workflow (primarily read-level, supports host removal and standardized reporting).  
**Engineering Features**: Nextflow + nf-core best practices; container/Conda; convenient for batch processing and reproducibility.

#### Core Dependency Ecosystem (by Function)
- **QC/Preprocessing**: FastQC/falco, fastp, AdapterRemoval2, Porechop/Porechop_ABI, Filtlong, Nanoq, BBDuk, PRINSEQ++  
- **Host Removal**: Bowtie2 (short reads), Minimap2 (long reads), Samtools  
- **Classifiers**: Kraken2, MetaPhlAn, Centrifuge, Kaiju, mOTUs, KrakenUniq, ganon, KMCP, DIAMOND, MALT, etc. (enabled by configuration)  
- **Post-processing/Visualization**: Bracken, Taxpasta, Krona, MultiQC, Nonpareil  

#### Read Compatibility (Conclusion)
- **Short Reads: Fully Compatible**.  
- **Long Reads: Fully Compatible**.

#### Usage Recommendations
- If the goal is "broad screening + rapid overview": prioritize taxprofiler;  
- If the goal is "high-confidence list": apply **consensus/threshold strategies** to taxprofiler output (see Chapter 7).

---

### 4.2 GOTTCHA2
**Positioning**: High-specificity classification based on "taxon-specific unique signature fragments (unique signatures)".  
**Core Concept**: Only reports when reads hit species/genus-specific regions, reducing false positives at the source.

#### Core Dependency Ecosystem
- **Alignment Engine**: Minimap2 (core in modern versions), (optional/legacy) BWA-MEM  
- **Post-processing**: Samtools, Python (Pandas, etc.)  
- **Database**: Pre-computed signature library (library construction scripts extract specific fragments from reference genomes)

#### Read Compatibility (Conclusion)
- **Short Reads: Compatible**  
- **Long Reads: Compatible** (Minimap2 is more friendly to high-error-rate long reads)  
- However, its "conservative" strategy means: **may miss extremely low abundance, database-missing, or highly variant targets**.

#### Usage Recommendations
- Very suitable as a "secondary confirmation" tool: perform signature coverage validation on key targets reported by taxprofiler/Kraken.

---

### 4.3 sourmash
**Positioning**: Sketch and containment retrieval based on MinHash/FracMinHash for rapid similarity/containment judgment.  
**Advantages**: Extremely fast, resource-light, suitable as a "coverage/consistency validation" and large-scale search component.

#### Core Dependency Ecosystem
- Primarily built-in implementation (Rust + Python); main modules: `sketch`, `gather`, `lca`, `tax`.  
- Does not depend on external aligners or assemblers (but often combined with "assembled contigs" to improve interpretability).

#### Read Compatibility (Conclusion)
- **Short Reads: Fully Compatible**  
- **Long Reads: Compatible**, but note: error rates reduce the proportion of matchable k-mers, thereby reducing intersect/containment metrics.  
- Practical recommendation: For long reads without error correction, recommend using sourmash as "coarse validation" rather than the sole evidence chain.

---

### 4.4 CLARK (including CLARK-S)
**Positioning**: Discriminative k-mer classification; CLARK-S uses spaced seeds to improve robustness to variation/errors.  
**Advantages**: Fast speed, good specificity for in-database targets.  
**Risks**: Strong dependency on database coverage; long read errors affect exact k-mer matching; CLARK-S often consumes more memory.

#### Core Dependency Ecosystem
- Core Engine: C++  
- Variants: CLARK, CLARK-S, CLARK-l  
- Database: Requires building/downloading corresponding reference libraries (library version significantly affects results)

#### Read Compatibility (Conclusion)
- **Short Reads: Fully Compatible**  
- **Long Reads: Usable but recommend CLARK-S**, and need to prepare sufficient memory and follow-up validation steps.

---

### 4.5 nf-core/mag
**Positioning**: Metagenomic assembly, binning, MAG quality assessment and annotation (reconstructing genomes, not aiming to "detect as many species as possible").  
**Key Value**: Can produce contigs, bins, and MAGs usable for evolutionary/functional analysis.

#### Core Dependency Ecosystem (Simplified Table)
- **Preprocessing**: fastp/AdapterRemoval, Porechop/Filtlong, FastQC  
- **Assembly**: MEGAHIT, (meta)SPAdes (can perform hybrid assembly)  
- **Binning/Integration**: MetaBAT2, MaxBin2, CONCOCT, DAS Tool, SemiBin2, etc.  
- **QC/Classification**: CheckM/CheckM2, GUNC, QUAST, GTDB-Tk, CAT, Prokka/Prodigal, BUSCO  
- **Coverage Calculation**: Bowtie2/Minimap2 + Samtools (for mapping/coverage)

#### Read Compatibility (Conclusion)
- **Short Reads: Fully Compatible**  
- **Hybrid Assembly (Short + Long): Friendly (workflow configurable)**  
- **Pure Long Reads: Can perform some steps, but assembler strategy may not be optimal (if pursuing pure long read assembly, dedicated long read assemblers are more suitable)**.

---

### 4.6 rvdb-viral-metagenome-nf (rvdb-viral)
**Positioning**: Discovery-oriented workflow for viral metagenomics: "assembly + RVDB homology (protein) search + (optional) viral feature validation".  
**Core Objective**: Improve discovery capability for environmental viruses/distant viruses, especially reconstruction and annotation of large genome viruses such as NCLDV.

#### Core Dependency Ecosystem (as described by actual workflow)
- **Assembly**: MEGAHIT, SPAdes, viralFlye/(MetaFlye-like)  
- **Gene Prediction**: Prodigal  
- **Homology Search**: DIAMOND (protein level)  
- **Database**: RVDB (viral reference database)  
- (Optional) Viral QC/Boundaries: CheckV, VirSorter2, etc. (whether enabled depends on workflow version/configuration)

#### Read Compatibility (Conclusion)
- **Short Reads: Compatible (assembly + annotation)**  
- **Long Reads: Compatible (more beneficial for long contigs and large virus reconstruction)**  
- **Hybrid Strategy: Suitable** (can improve reconstruction quality through short read accuracy + long read connectivity).

---

### 4.7 MLMVD-nf
**Positioning**: High-specificity viral mining/screening workflow centered on "multi-tool consensus" (more like a "strict filter").  
**Core Concept**: Multi-source evidence voting from VirSorter2, DeepVirFinder, viralFlye, etc.; outputs smaller but more reliable candidate sets, suitable for subsequent experimental validation.

#### Core Dependency Ecosystem (Representative)
- Assembly: MEGAHIT/SPAdes/viralFlye (for candidate contigs)  
- Viral Features: VirSorter2, DeepVirFinder (machine learning/features)  
- Optional: CheckV, hallmark gene statistics, structural features, etc.

#### Read Compatibility (Conclusion)
- **Short Reads: Compatible**  
- **Long Reads: Compatible**.  
---

### 4.8 KrakenMetaReads-nf
**Positioning**: Author-developed "assembly before classification" Kraken2 ecosystem wrapper workflow (contig-level classification is core).  
**Value**: Uses long contigs to reduce LCA merging and short fragment ambiguity; particularly suitable for species-level positioning of giant viruses/large genome fragments.

#### Core Dependency Ecosystem
- Assembly: MEGAHIT, SPAdes, (long reads) Flye/viralFlye  
- Classification: Kraken2, Bracken  
- Visualization: Krona  
- (Optional) QC: FastQC/MultiQC, filtering steps, etc.

#### Read Compatibility (Conclusion)
- **Short Reads: Compatible (classification after assembly)**  
- **Long Reads: Compatible (long read assemblers like Flye produce long contigs)**  
- Risk: Low-abundance taxa may be lost during assembly stage.

---

## 5. Empirical Results Analysis (Two Samples)

### 5.1 Analysis Framework and Naming Conventions (Avoiding Conceptual Confusion)
This chapter categorizes the 8 runnable objects into four strategy types:  
1) **Direct read classification**: taxprofiler (Kraken2/Bracken modules, etc.), (optional) direct read input for Kraken2/CLARK/GOTTCHA2;  
2) **Contig classification after assembly**: KrakenMetaReads-nf, (auxiliary) classification interpretation of contigs/MAGs from nf-core/mag;  
3) **Homology (protein/alignment) evidence chain**: rvdb-viral (DIAMOND/RVDB), etc.;  
4) **Viral feature/consensus screening**: MLMVD-nf (multi-tool consensus).


### 5.2 taxprofiler (MetaTaxProfiler) Key Observations
#### 5.2.1 Short Read Sample `llnl_66ce4dde`
- **Broad-spectrum Detection**: Read-level classification can form a "long-tail distribution", capturing a large number of low-abundance signals.  
- **Risk Point**: Taxa with "reads < 10" in the long tail are more affected by database bias, contamination, and short fragment homology, and must be combined with thresholds and coverage evidence (Chapter 7).  

### 5.3 KrakenMetaReads-nf Key Observations
#### 5.3.1 Short Read Sample `llnl_66ce4dde`
- **MEGAHIT vs SPAdes**: SPAdes often produces more continuous viral contigs (beneficial for subsequent evidence chains); MEGAHIT is more "comprehensive" and may retain more low-abundance fragments.  
- **Value of Consensus Contigs**: Contig sets overlapping between the two assemblers are more robust and suitable as "high-confidence candidate sets" for subsequent validation (DIAMOND / VirSorter2 / CheckV).

#### 5.3.2 Long Read Sample `llnl_66d1047e`
- **Long Read Assembly (Flye/viralFlye) Advantages**: Easier to obtain contigs from tens of kb to hundreds of kb, providing strong evidence chains for large viruses such as NCLDV.  
- **Classification Interpretation More "Genome-like Evidence"**: When long contigs receive clear classification in Kraken2, their credibility is usually significantly higher than scattered hits from short reads (but still requires coverage distribution and gene feature validation).

### 5.4 rvdb-viral Key Observations (Discovery-Oriented Evidence Chain)
- **Consensus Set (Dual-Track Overlap)**: Contig sets overlapping in dual tracks (assembly/features/homology) are more reliable; single-path hits require caution.  
- **NCLDV Clues**: In long read samples, if there are numerous protein hits related to Mimiviridae/Phycodnaviridae/Marseilleviridae, etc., combined with long contigs and gene features, can form strong evidence chains supporting "NCLDV enrichment".  
- **Boundaries of Abundance Interpretation**: Metrics such as RPM/RPKM are affected by mapping strategies, repetitive sequences, and uneven coverage; "high RPM" indicates dominance but cannot be used alone for "active infection/epidemiological conclusions", requiring sample background and time series support.

### 5.5 MLMVD-nf Key Observations (High-Specificity Screening)
- **3-tool consensus / 2-tool consensus**: Strict consensus converges candidate sets to a small number of contigs, suitable as "priority lists" for subsequent experimental validation.  
- **Cost of Strict Strategy**: May miss real but incompletely evidenced viral contigs (especially short, low-coverage, or database-missing taxa).  
- **Recommendation**: Treat MLMVD-nf as a "confirmation candidate set generator", not a "broad-spectrum detector".

### 5.6 Role of GOTTCHA2 / CLARK / sourmash in This Assessment (Positioning Description)
- **GOTTCHA2**: Suitable for signature coverage confirmation of "key targets"; may be more conservative for distant/low-abundance targets.  
- **CLARK (especially CLARK-S)**: Suitable for rapid screening of in-database covered targets; valuable as one of the "confirmatory evidence".  
- **sourmash**: Suitable for "coverage-style validation" using intersect_bp/containment, more effective for assembled contigs.

---

## 6. Comprehensive Comparison and Key Metrics Assessment

### 6.1 Sensitivity vs. Specificity Trade-off (Default Output vs. Consensus/Threshold)
| Tool/Workflow | Default Output Sensitivity | Default Output Specificity | Specificity Improvement Potential After Consensus/Threshold | Notes |
|---|---|---|---|---|
| taxprofiler | High | Medium | High | Broad coverage; requires thresholds/coverage/multi-evidence chains to reduce false positives |
| GOTTCHA2 | Medium-Low | High | Medium | Signature strategy conservative, suitable for confirmation |
| sourmash | Medium | Medium-High (high when thresholds sufficient) | Medium | Suitable for coverage/consistency validation |
| CLARK (CLARK-S) | Medium | High | Medium | Strong for in-database targets; recommend CLARK-S for long reads with validation |
| nf-core/mag | (Not targeting detection) | (Not separately evaluated) | High (combined with MAG QC) | Core value in MAG reconstruction and quality evidence chain |
| rvdb-viral | High | Medium | High (using consensus set) | Strong discovery capability; consensus sets more reliable |
| MLMVD-nf | Low | Very High | (Already reflected in strategy) | Consensus filter: trades sensitivity for specificity |
| KrakenMetaReads-nf | Medium | Medium | Medium-High | Assembly-before-classification reduces ambiguity but may lose low-abundance taxa |

### 6.2 Computational Resources and Engineering Costs (Empirical Conclusions)
- **Resource-Intensive (CPU/Time)**: nf-core/mag, rvdb-viral, MLMVD-nf (primarily assembly/multi-tool inference).  
- **Memory-Intensive**: Kraken2/CLARK-type tools depend on database indices (memory strongly correlated with library size).  
- **Lightweight**: sourmash (sketch extremely small, suitable for large-scale search and rapid validation).  
- **Engineering Maturity**: nf-core series and Nextflow workflows (taxprofiler, nf-core/mag, and self-developed NF workflows) are more suitable for production in terms of batch execution, logging, and checkpoint resumption; Standalone tools (CLARK/GOTTCHA2/sourmash) are more suitable for point-to-point validation or small-scale tasks.

### 6.3 Database is the "Invisible Deciding Factor"
- RefSeq/standard libraries often underestimate environmental virus and phage diversity;  
- RVDB is more friendly to viruses (especially environmental viruses and distant sequences), but database annotation bias should also be guarded against.


---

## 7. False Positive Control Strategies: The Lifeline of Viral Detection

Common sources of false positives in metagenomic viral detection: host contamination, reagent contamination (kitome), short fragment homology, database misannotation, low-complexity sequences and repetitive sequences, etc. Recommend adopting layered evidence chains:

### 7.1 Minimum Essential Strategy (Strongly Recommended)
1. **Host Removal (Host depletion)**: Use Bowtie2 for short reads; Minimap2 for long reads.  
2. **Threshold Filtering**: Set minimum read count / relative abundance thresholds for read-level classification results (thresholds should be adjusted with data volume).  

### 7.2 Recommended "Multi-Evidence Chain Consensus"
- **Strategy A: Multi-Tool Consensus (≥2 tools support)**: For example, taxprofiler's Kraken2 + Kaiju/DIAMOND (if enabled) and/or GOTTCHA2/CLARK support.  
- **Strategy B: Assembly Evidence Priority**: For important findings, prioritize requiring at least one medium-to-long contig (>5–10kb, adjusted according to virus type) with:  
  - Viral hallmark genes (such as MCP, DNApol, helicase, etc., depending on taxon)  
  - Reasonable gene density and directionality  
  - Support from DIAMOND/RVDB or VirSorter2/CheckV evidence  

### 7.3 Interpretation Principles for "Single Tool, Low Abundance, Short Fragment Hits"
- Items appearing only in a single tool, with very few reads, and lacking coverage/assembly evidence should be labeled as:  
  **"Low Confidence Candidate (Requires Validation)"**, avoiding direct entry into public health decision-making or epidemiological inference.

---

## 8. Decision Matrix (Scores 1–5)

> **Scoring Criteria**: 1 = Weak/Unsuitable, 3 = Medium/Usable, 5 = Strong/Very Suitable.  
> **Scoring Framework**: Default based on "target scenarios and reproducible deployment of this report"; if strict consensus/thresholds are applied, specificity and reliability can be additionally improved (see Chapter 7).

| Dimension | taxprofiler | rvdb-viral | MLMVD-nf | nf-core/mag | GOTTCHA2 | CLARK | sourmash | KrakenMetaReads-nf |
|---|---:|---:|---:|---:|---:|---:|---:|---:|
| Known Virus Sensitivity (Default) | 5 | 4 | 2 | 2 | 3 | 3 | 3 | 3 |
| Distant/Environmental Virus Discovery Capability | 3 | 5 | 3 | 4 | 1 | 1 | 2 | 3 |
| Specificity (Default) | 3 | 3 | 5 | 4* | 5 | 4 | 4 | 3 |
| Result Reliability (Interpretable Evidence Chain) | 4 | 4 | 4 | 5 | 4 | 3 | 4 | 4 |
| Analysis Speed | 3 | 1 | 2 | 1 | 4 | 5 | 5 | 3 |
| Memory Efficiency | 2 | 3 | 3 | 2 | 4 | 2 | 5 | 2 |
| Genome Reconstruction Capability | 1 | 5 | 2 | 5 | 1 | 1 | 1 | 4 |
| Engineering Maturity (HPC Reproducible) | 5 | 5 | 4 | 5 | 3 | 3 | 4 | 5 |

\* nf-core/mag's high specificity score primarily comes from evidence chains provided by MAG QC (CheckM/GUNC/coverage, etc.), not read-level classification.

---

## 9. Recommended Solutions

### 9.1 Solution One: High-Confidence Consensus and Benchmark Analysis (High-Confidence Consensus)
**Recommendation**: taxprofiler (primary screening) + (key targets) GOTTCHA2 / sourmash / assembly evidence  
**Logic**:  
- taxprofiler for "broad-spectrum detection + rapid overview";  
- Apply to key entries: host removal + thresholds + coverage/assembly validation;  
- When "confirmation level" is needed, use GOTTCHA2 for signature coverage validation; use sourmash/mapping coverage for auxiliary judgment.  
**Applicable**: Wastewater environmental virus monitoring, pre-clinical screening projects.

### 9.2 Solution Two: Emerging/Outbreak Pathogen Discovery (Discovery & Outbreak)
**Recommendation**: rvdb-viral (discovery and assembly) + MLMVD-nf (strict candidate set) + (optional) nf-core/mag (in-depth reconstruction and ecology/host association)  
**Logic**:  
- First use rvdb-viral for assembly and protein homology search to maximize discovery capability;  
- Use MLMVD-nf for strict screening of "weak homology/dark matter candidates";  
- If host association and genome-level analysis are needed, proceed to nf-core/mag.  
**Applicable**: Environmental unknown virus surveys, outbreak tracing requiring rapid production of high-quality contig/genome evidence chains.

### 9.3 Solution Three: Wastewater Virus Monitoring (Wastewater Surveillance)
**Recommendation**: taxprofiler (batch screening) → (suspected positive targets) GOTTCHA2 confirmation → (key samples) KrakenMetaReads-nf / rvdb-viral for assembly validation  
**Logic**: Wastewater has high background noise and contamination, must use "multi-evidence chains + coverage thresholds + secondary confirmation".  
**Output Recommendation**:  
- Clearly distinguish "screen-positive" from "confirm-positive" in reports.

### 9.4 Solution Four: Large-Scale Environmental/Sewage Monitoring (Ultra-Diverse Samples)
**Recommendation**: sourmash (rapid similarity/containment coarse screening + deduplication) + taxprofiler (core sample fine screening) + (key samples) nf-core/mag/rvdb-viral deep mining  
**Logic**: First use lightweight tools to reduce scale, then invest heavy-resource workflows on key samples.

### 9.5 Final Recommendations (Decision-Oriented)
1. **Treat "Deployment Reproducibility" as the Primary Prerequisite**: Tools that cannot run stably should not enter public health decision chains.  
2. **Never Use Single Read Count as "Existence Conclusion"**: Must use coverage/assembly/multi-tool consistency.  
3. **Establish "Evidence Tier Labels" for Key Findings**: Screen-positive / Supportive / Confirmed.  

---

## 10. Appendix

### 10.1 Terms and Explanations (Simplified)
- **Sensitivity**: More inclined to "try not to miss detections";  
- **Specificity**: More inclined to "try not to false alarm";  
- **Consensus/Threshold Output**: Result set after applying rules to default results (usually increases specificity, decreases sensitivity);  
- **NCLDV**: Nucleocytoviricota (Nucleocytoplasmic Large DNA Viruses);  
- **Contig**: Continuous sequence fragments obtained from assembly;  
- **MAG**: Metagenome-Assembled Genome (genome draft after binning).  

---

**End of Report**

