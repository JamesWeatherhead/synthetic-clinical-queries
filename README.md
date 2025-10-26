# Synthetic Clinical Queries
Reproducible pipeline and benchmark dataset for generating synthetic PHI-containing clinical queries to evaluate HIPAA-compliant de-identification in AI workflows

**Why This Framework Matters**

There are currently a number of papers that have been written showcasing Large Language Model's ability to generate realistic synthetic data: examples here cited with [1], [2], [3], etc. However, these synthetic datasets are mainly EHR's, soap notes, etc. These sorts of datasets are used in deidentification studies to find patient health information (PHI) and replace them with <PLACEHOLDER> like "... MRN: 673823425" is replaced to <MRN> to redact the PHI. However, in the case of testing these sorts of synthetic datasets containing PHI, one could argue: over-redacting is somewhat of a success in ensuring ultimate privacy. However, this repo is about a new kind of synthetic data. A kind of synthetic data that has a fundamental trade-off in that redacting it you cannot optimize for both HIPAA compliance AND utility of the data itself. **This dataset is specifically for exploring how to optimize both HIPAA compliance AND utility—other PHI synthetic datasets are mainly for HIPAA compliance alone, but search queries require preserving context for utility, so there's an inherent trade-off.** This synthetic dataset is of Physicians (or other health professionals) querying Large Language Models for clinical assistance relevant to a patient's specific context.

Business Associate Agreement (BAA) HIPAA-compliant Large Language Models (LLMs) can process PHI data but lack internet search today, because adding search risks PHI leakage to external APIs [5]. Physicians are increasingly using AI assistants with patient-specific context in their queries, to enhance the tailoring of the output to the patient [6]. In order to turn on a de-identification system to enable internet search for BAA HIPAA LLMs, we need deterministic de-identification systems that guarantee zero PHI transmission, as violations at a tier 1 level are $141 to $35,581 per violation which is for lack of knowledge, tier 2 is $1,424 to $71,162 per violation for reasonable cause, tier 3 is $14,232 to $71,162 per violation for willful neglect that is corrected within the required time period, and tier 4 is $71,162 per violation for willful neglect that is not corrected within the required time period [4]. So even if we have solutions that work 99.9% of the time, 0.1% of millions of queries is still thousands of leaks, which is unacceptable.

With this all in mind, obtaining real physician queries containing PHI is virtually impossible: IRB approval is prohibitively complex for this specific case [7], commercial BAA HIPAA-compliant LLM providers like Qualified Health can't share physician queries, and manual creation is slow, expensive, and unscalable. Also, I would like to note, that this dataset addresses Shadow AI use, whereby health care professionals are using AI without authorization and exposing PHI to non-compliant systems and thus, trying to ask them to reproduce this sort of a workflow would: A) be impossible as how do we identify those who use Shadow AI without firing them, B) unethical [8].

This framework solves these issues by enabling the generation of labeled synthetic queries where every PHI element has ground-truth annotations. This isn't just a benchmark - it's an ethical proxy that advances research in sensitive domains where real data is legally and ethically inaccessible. Developers can trace PHI through Model Context Protocol (MCP) server implementations, multi-stage agentic pipelines, and external search api integrations - verifying complete HIPAA compliance at each stage. This framework enables researchers to build confident deployments of HIPAA-compliant AI systems that can safely access current medical knowledge through MCP servers and internet search through which they don't have a BAA with.

These queries are not meant to be ground truth for realism, they're meant to be ground truth for the start which is simple ground truth detection. As every medical professional prompts uniquely, simulating all of them in one dataset may need to be tailored to the organization, which I have accounted for, by allowing the institution to change the few-shot examples in the synthetic_data_gen_method.ipynb.

This dataset is the start of a much larger discussion about HIPAA compliance with LLMs. Currently it is considered "enough" to be HIPAA compliant if you sufficiently de-identify at the individual query level [9]. However, as multiple messages are sent in succession in the same chat, with each message containing more and more de-identified context about the patient, that begins to narrow down who the person could be from a large amount to a very small amount. This is known as the k-anonymity re-identification issue [10]. As each term in a query has a probability within a population. For example, roughly 48% of people in the United States have hypertension [11], so querying for: "What are the latest treatment options for a 55 year old male with Hypertension?" is not breaking HIPAA there are probabilities within that query. HIPAA according to the Safe Harbor method (45 CFR 164.514(b)(2)) states you only have to remove age older than 89 [12]. So that query alone, is considered to be sufficiently de-identified. However, what if the next query is: "And what if he also has Type 2 Diabetes?". Well now, Type 2 Diabetes has a probability of about 11.6% [13]. And the follow up query is "And he's on dialysis for CKD stage 5" which has probabilities of about 0.2% or less for dialysis-dependent CKD [14], which you have now re-identified the patient inadvertently, because only a handful of patients within a hospital system have those sets of things.

## Dataset Statistics
![PHI Density Distribution](data/figures/figure2_query_complexity_distribution.png)
*Distribution of PHI elements per query, showing a mix of densities for robust testing.*

![PHI Type Distribution](data/figures/figure1_phi_type_distribution.png)
*Breakdown of PHI types in the dataset, emphasizing common HIPAA identifiers like names and dates.*

## Data Format Example
![Example Clinical Query](data/figures/figure0_example_query.png)
*Sample query with ground-truth PHI tags.*

## The Fundamental Trade-Off
![Recall and Leaked Entities](data/AWS-Comprehend-Medical-DetectPHI/Figures/Figure2_Positive_HIPAA_Compliance.png)
*Results for queries containing PHI run through AWS Comprehend Medical, illustrating the trade-off for HIPAA compliance (recall vs. leaked entities).*

![Specificity and False Positives](data/AWS-Comprehend-Medical-DetectPHI/Figures/Figure3_Negative_Utility_Preservation.png)
*Results for queries without PHI run through AWS Comprehend Medical, showing over-redaction risks and utility preservation (specificity vs. false positives).*

- [1]: https://arxiv.org/abs/2508.08529
- [2]: https://pmc.ncbi.nlm.nih.gov/articles/PMC11836953/
- [3]: https://pmc.ncbi.nlm.nih.gov/articles/PMC11512648/
- [4]: https://www.hipaajournal.com/hipaa-violation-fines/
- [5]: https://www.hhs.gov/hipaa/for-professionals/special-topics/cloud-computing/index.html
- [6]: https://www.ama-assn.org/practice-management/digital-health/what-doctors-wish-patients-knew-about-using-ai-health-tips
- [7]: https://www.hhs.gov/hipaa/for-professionals/special-topics/research/index.html
- [8]: https://aihc-assn.org/importance-of-addressing-shadow-ai-for-hipaa-compliance/
- [9]: https://www.hhs.gov/hipaa/for-professionals/privacy/guidance/de-identification/index.html
- [10]: https://pmc.ncbi.nlm.nih.gov/articles/PMC2528029/
- [11]: https://www.cdc.gov/high-blood-pressure/data-research/facts-stats/index.html
- [12]: https://www.hhs.gov/hipaa/for-professionals/privacy/special-topics/de-identification/index.html
- [13]: https://www.cdc.gov/diabetes/php/data-research/index.html
- [14]: https://www.niddk.nih.gov/health-information/health-statistics/kidney-disease