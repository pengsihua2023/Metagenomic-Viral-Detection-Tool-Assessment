# Meeting Minutes: Viral Genome Analysis Pipeline Evaluation

**Date:** 12:00PM, 12/18/2025   
**Meeting Type:** Evaluation Meeting  
**Participants:** Bahl,Justin; Cornforth, Daniel; Lorentz, Benjamin; Barnes, John R; Goraichuk, Iryna; Caravas, Jason; Taibl, Kaitlin; and Peng, Sihua.  
**Duration:** 1 Hour  

---

## Executive Summary

This meeting focused on the evaluation of viral genome analysis pipelines and workflows for wastewater surveillance applications. The team discussed evaluation results, recommended workflows, database selection criteria, and next steps for transitioning from simulated to real-world data analysis.

---

## 1. Pipeline Evaluation Overview

### 1.1 Evaluation Scope
- **Total pipelines evaluated:** 10 pipelines were initially recommanded by CDC.
- **Pipelines fully evaluated:** 5 pipelines (5 were not evaluated beyond initial assessment due to HPC cluster permission requirements)
- **In-house developed pipelines:** 3 workflows (developed by Peng during the project)
- **Evaluation duration:** Less than 4 months of work

### 1.2 Key Findings
- No single pipeline was found to work best for all scenarios
- **Best sensitivity:** TaxProfiler demonstrated the highest sensitivity
- **Best specificity:** GOTTCHA2 demonstrated the best specificity, achieving high scores in the resulting output files
- **Best contig assembly:** MEGHIT showed the best performance for contig assembly
- **Best classification:** Kraken2 performed best for classification based on contigs after assembly
- **Best machine learning method:** MLMVD-nf (Machine Learning Method) developed in-house showed strong performance

---

## 2. Recommended Workflows

The team recommended **three workflow solutions** that combine multiple pipelines rather than relying on a single tool:

### Solution 1: TaxProfiler + GOTTCHA2
- **Purpose:** Benchmark analysis and monitoring
- **Rationale:** 
  - TaxProfiler provides high sensitivity for initial screening
  - GOTTCHA2 provides confirmation with high specificity
  - Suitable for monitoring known viruses

### Solution 2: rvdb-viral-metagenome-nf + MLMVD-nf
- **Purpose:** Discovery of unknown pathogens
- **Rationale:**
  - Both tools developed in-house
  - rvdb-viral-metagenome-nf uses viral database for discovery
  - MLMVD-nf (Machine Learning Method) helps filter through noise
  - Suitable for scenarios where the pathogen is unknown 

### Solution 3: Extended Water Monitoring
- **Purpose:** Horizon scanning/screening approach
- **Components:**
  - First step: TaxProfiler for initial screening
  - Second step: GOTTCHA2 for confirmation
  - Third step: Assembly validation with rvdb-viral-metagenome-nf
- **Rationale:** Combines database advantages with tracking capabilities

---

## 3. Evaluation Metrics and Decision Matrix

### 3.1 Evaluation Criteria
- **Sensitivity:** Ability to detect true positives
- **Specificity:** Ability to avoid false positives
- **Reliability:** Based on contig results and error rates in output genomes
- **Speed:** All workflows completed within 20 hours (some in 2-3 hours)
- **Memory requirements:** Maximum 300 GB

### 3.2 Decision Matrix
- The decision matrix was developed based on simulation dataset results
- Scoring system: 1-5 scale (qualitative assessment in some cases)
- Factors considered: sensitivity, specificity, and viral discovery capabilities

---

## 4. Database Selection and Maintenance

### 4.1 Databases Used
- **TaxProfiler:** Kraken2, viral databases
- **GOTTCHA2:** Rough, thick, bacteria database (~100 GB) - most comprehensive database
- **rvdb-viral-metagenome-nf:** RVDB Viral database from NCBI
- **Other databases:** Various viral databases from NCBI and other sources

### 4.2 Database Selection Criteria
- **Size preference:** Databases no larger than 10 GB for easier updates
- **Update frequency:** Preference for databases updated within the current year (2025)
- **Exclusion criteria:** Databases not updated for 3+ years will not be selected
- **Custom databases:** Most software packages allow users to create custom databases

### 4.3 Maintenance Challenges
- **Concerns raised:**
  - Database updates may invalidate past results
  - Need to determine update frequency
  - Different workflows may require different database update schedules
  - Some databases are not updated frequently
- **Future work:** Peng will conduct research on database selection and evaluation, reporting findings in the future

---

## 5. Transition to Real-World Data

### 5.1 Data Source
- **Source:** real wastewater samples
- **Data type:** Untargeted metagenomic sequencing data


### 5.2 Analysis Approach
- **Initial approach:** Use discovery pipeline (Solution 2) to identify what's present
- **Subsequent steps:** Develop more reference-based and targeted approaches based on findings
- **Known targets in samples:**
  - SARS-CoV-2 (expected to be present)
  - Measles (detected early, low abundance)
  - H5 influenza (detected in low abundance last year)
  - Environmental contaminants

### 5.3 Challenges with Real Data
- No spike-ins or known references for validation
- Need to distinguish between true pathogens and environmental contaminants
- Understanding background noise in wastewater samples
- Determining what constitutes a novel virus vs. an error

---

## 6. Performance Characteristics

### 6.1 Computational Requirements
- **Runtime:** All workflows complete within 20 hours (some in 2-3 hours)
- **Memory:** Maximum 300 GB required
- **Parallelization:** Software supports parallelization, but optimization was not the focus of initial evaluation
- **Sample size:** Evaluation performed on single samples

### 6.2 Limitations
- Some pipelines require HPC cluster permissions (violated usability principle)
- Evaluation based on simulated data (real data evaluation pending)
- Some assessments are qualitative rather than quantitative

---

## 7. Deliverables and Next Steps

### 7.1 Final Report
- **Format:** PDF document (detailed version preferred)
- **Timeline:** February, 2026
- **Contents:**
  - Detailed decision matrix with empirical results
  - Test results and outputs
  - All outputs will be available on GitHub repository
  - Database selection recommendations
  - Workflow documentation

### 7.2 Immediate Next Steps
1. Complete real-world data analysis using Secure Bio samples
2. Evaluate workflows on real data
3. Conduct database selection research
4. Prepare detailed final report
5. Schedule final closing meeting in January

### 7.3 Future Considerations
- Potential publication: Would require additional months of work to mature analyses
- User testing: May require period of user testing to refine database recommendations
- Community feedback: Need input from wider community on database selection

---

## 8. Key Discussion Points

### 8.1 Tool Convergence
- GOTTCHA2 is becoming a preferred tool (also used by UD folks and Mark Johnson)
- Convergence on utility of certain tools observed

### 8.2 In-House Development
- **Pipelines 6, 7, and 8:** Developed by Peng during the project
- These are workflows combining existing tools, not entirely new software
- Distinction made between "pipelines" (individual tools) and "workflows" (combining multiple pipelines)

### 8.3 Scope and Expectations
- **Project duration:** Less than 4 months
- **Project goal:** Help orient the team in a space they haven't been working in extensively
- **Outcome:** Provides a good starting point, though not comprehensive
- **Feedback:** Team satisfied with progress and deliverables

---

## 9. Action Items

| Action Item | Owner | Timeline |
|------------|-------|----------|
| Complete real-world data analysis | Peng | ASAP |
| Conduct database selection research | Peng | Future |
| Prepare detailed final report with empirical results | Team | February |
| Schedule final closing meeting | Team | January |
| Provide list of potential public health targets in samples | Jason/Team | Follow-up |
| Coordinate data access for real samples | Team | Follow-up |

---

## 10. Open Questions and Concerns

1. **Database maintenance:** How to manage long-term database updates without invalidating past results?
2. **Real data validation:** How to validate results on real data without known references?
3. **Background noise:** Understanding typical wastewater composition and fluctuations
4. **Novel virus detection:** Distinguishing novel viruses from errors in real-world scenarios
5. **Customization:** Balancing general-purpose workflows with scenario-specific customization needs

---

## 11. Meeting Conclusion

The meeting concluded with:
- Confirmation that the project is on track and meeting expectations
- Agreement on next steps for real-world data analysis
- Confirmation of final report format and timeline
- Plans for a follow-up meeting in January

**Next Meeting:** Final closing meeting scheduled for January [date TBD]

---

## Appendix: Technical Notes

- **GitHub repository:** All outputs and results will be available on GitHub
- **Communication challenges:** Some technical difficulties with communication during the meeting
- **IT issues:** Some participants experienced connectivity issues
- **Presentation format:** Slides available (referenced slides 23, 24, 25 for recommended workflows)

---

**Prepared by:** Sihua Peng  
**Date:** 12/23/2025  













