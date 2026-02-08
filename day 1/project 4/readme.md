# P4 – Simple Evaluations

## Overview

Simple workflow to **demo n8n’s Evaluation nodes**: it pulls rows from a dataset (users upload `evals.csv` into an n8n Data Table), runs an LLM classification, then records metrics + outputs back to the evaluation run.  

## Setup

* In n8n, create a **Data Table** (e.g. “Evals”) and **upload `evals.csv`** as the dataset. 

## Flow (high level)

1. **When fetching a dataset row** (Evaluation Trigger)

   * n8n feeds one row at a time from the uploaded Data Table.

2. **Basic LLM Chain + Google Gemini Chat Model**

   * Classifies the ticket text as `Urgent` or `Not urgent` (structured output).

3. **Evaluation (check if evaluating)**

   * Ensures the workflow is running in evaluation context.

4. **Evaluation1 (setMetrics)**

   * Compares **expected_output** (from the dataset row) vs the model’s actual label and stores a `categorization` metric.

5. **Add Output**

   * Writes outputs back to the evaluation run (model output + metric fields).
