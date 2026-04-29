# Accelerating-Clinical-Codelist-Generation-for-NICE-using-AI
April 2026

*Darya Yarparvar; Olga Shapovalova; Kseniia Warren; Abdullah Sheikh; Eshwar Meduri; Joe Flanagan;*


## Problem Statement
The National Institute for Health and Care Excellence (NICE) requires highly accurate, consistent, and auditable clinical codelists, typically SNOMED CT codes, to identify patient populations for healthcare research and quality indicators. Currently, this process is manual, time-consuming, inconsistent across analysts, difficult to audit, and prone to obsolescence due to terminology changes. This inefficiency hinders timely and robust healthcare research.  
This project introduces an automated, Agentic Retrieval-Augmented Generation (RAG) pipeline as an AI co-pilot for clinical analysts, automating codelist creation to improve efficiency, consistency, and adaptability to evolving terminologies and reducing the manual process.

## Suggested Solution
We built an Agentic Retrieval-Augmented Generation (RAG) pipeline that acts as an AI co-pilot for clinical analysts (Fig. 1). The system is fully local, open-weight, and audit-ready, satisfying NICE's data governance requirements.

<img width="989" height="821" alt="image" src="https://github.com/user-attachments/assets/d453350c-ebdb-4f1f-a541-8d8948179d09" />

Figure 1. An overview of the suggested Agentic RAG pipeline solution for accelerating clinical codelist generation. The pipeline comprises four sequential stages: a query router that classifies the research question by clinical entity type; a ReAct agent loop that iteratively retrieves and refines results; hybrid retrieval with graph expansion combining sparse and dense embeddings fused via RRF across PCD and SNOMED CT; LLM-based code generation using Phi-4 to classify each candidate code.

