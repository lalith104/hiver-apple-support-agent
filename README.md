# AI Customer Support Agent for AppleSupport

## 1. Project Overview

This project builds an AI customer support agent for AppleSupport using historical customer-support conversations from Twitter.

The system performs three main tasks:

1. Classifies an incoming customer message into a small set of support intents.
2. Retrieves similar historical AppleSupport cases and uses their responses as evidence for drafting a reply.
3. Decides whether the issue should be **AUTO-HANDLE** or **ESCALATE** to a human, with a reason.

The system was designed with a risk-first approach: sensitive, unclear, or weakly supported issues are escalated rather than automatically answered.

## 2. Dataset

The project uses the Customer Support on Twitter dataset from Kaggle:

https://www.kaggle.com/datasets/thoughtvector/customer-support-on-twitter

The full dataset contains millions of tweets and multi-turn customer-support conversations.

Only the AppleSupport conversations were used for this project.

The full dataset is not included in this repository because of its size. To reproduce the notebook, download the dataset and place `twcs.csv` in the same directory as the notebook.

## 3. Approach

### Conversation Extraction

Customer messages were paired with AppleSupport replies using the tweet-response relationships in the dataset.

### Intent Classification

The initial intent classifier uses TF-IDF features and Logistic Regression.

The final intent layer also uses explicit keyword rules for high-risk or important categories.

The intent categories are:

* Software / iOS Issues
* Device / Hardware Issues
* Battery / Power
* Calls / Messaging
* Connectivity
* Apple Services / Apps
* Apple ID / Security
* Payment / Billing
* Other / Unclear

### Historical Retrieval

For each incoming message, the system retrieves the top three similar historical customer cases using TF-IDF cosine similarity.

The similarity scores are used as an evidence signal.

### Evidence Quality

Evidence is categorized as:

* STRONG
* MODERATE
* WEAK

### Routing

The system uses a risk-first routing policy.

Sensitive categories such as Payment / Billing and Apple ID / Security are escalated.

Other unclear issues are also escalated.

Clear issues with sufficient historical evidence can be AUTO-HANDLED.

## 4. Final Agent Output

For every customer message, the agent produces:

* Intent
* Similarity score
* Evidence quality
* Draft reply
* AUTO-HANDLE / ESCALATE decision
* Reason for the decision

Example:

**Customer message:**

> My iPhone battery is draining very fast

**Example output:**

```text
Intent: Battery / Power
Evidence quality: STRONG
Decision: AUTO-HANDLE
Reason: Clear intent with sufficient historical evidence.
```

## 5. Evaluation

A golden evaluation set of 200 customer messages was created and manually labelled.

### Final Results

| Metric                      |   Result |
| --------------------------- | -------: |
| Intent Accuracy             |    28.0% |
| Intent Macro F1             |    36.8% |
| Decision Accuracy           |    68.0% |
| Decision Macro F1           |    57.9% |
| Human Reply Quality         | 3.28 / 5 |
| Evidence >= Moderate        |    92.5% |
| LLM Relevance Within ±1     |    70.0% |
| LLM Grounding Within ±1     |    55.0% |
| LLM Actionability Within ±1 |    75.0% |
| LLM Safety Within ±1        |    15.0% |

## 6. Baselines

### Trivial Baseline

The trivial baseline uses a simple baseline strategy.

**Accuracy:** 34.5%
**Macro F1:** 25.7%

### Simple Retrieval Baseline

The simple retrieval baseline uses TF-IDF similarity to retrieve the closest historical customer case.

**Accuracy:** 65.5%
**Macro F1:** 39.6%

The final system adds intent classification, evidence quality assessment, and risk-based escalation.

## 7. Failure Analysis

### 1. Software vs Hardware Confusion

Messages about freezing, lag, restarting, or update-related problems can contain device words such as iPhone, Mac, or screen. This can cause software issues to be classified as hardware issues.

### 2. Generic or Unclear Messages

Messages such as "fix this" or "WTF" provide very little information. These cases are difficult to classify without additional context.

### 3. Apple Services vs General Issues

Messages mentioning apps, music, or Apple products can overlap between service problems and general software or device problems.

### 4. Lexical Retrieval Limitations

TF-IDF retrieval is mainly based on word overlap. A message can therefore retrieve a case with similar words but a different underlying problem.

### 5. Sensitive Issues

Payment, account, and security-related messages can resemble ordinary technical problems. The system therefore intentionally escalates these cases.

## 8. What Is Misleading About My Headline Number?

The 68.0% decision accuracy should not be interpreted as proof that the agent is production-ready.

The evaluation set contains only 200 examples, and the intent labels were created using a manually reviewed taxonomy with some rule-assisted labelling.

The intent classifier also has relatively low performance, with 28.0% accuracy.

The evidence score is based on TF-IDF similarity rather than true semantic understanding.

The LLM-as-judge was run using a small local model, so its agreement with human ratings is limited.

Therefore, the headline routing number is useful as an initial prototype result, but it does not establish production-level reliability.

## 9. What Was Not Built

The following were intentionally not built:

* Production API
* Web interface
* Persistent conversation memory
* Human support dashboard
* Fine-tuned language model
* Large-scale production deployment
* Automated access to private customer information
* Full-dataset production inference

The focus was on building and evaluating a reproducible prototype.

## 10. One-Week Improvement Plan

### Day 1–2

Create a cleaner human-labelled training set with more examples for each intent.

### Day 3

Replace keyword-heavy intent rules with a stronger semantic classifier.

### Day 4

Improve retrieval using embeddings instead of only TF-IDF.

### Day 5

Add an LLM-based grounded reply generator using retrieved historical cases as evidence.

### Day 6

Expand the golden evaluation set and perform more detailed failure analysis.

### Day 7

Run end-to-end evaluation, tune escalation thresholds, and prepare a production-style API.

## 11. Decision Log

### Decision 1: AppleSupport Was Selected

AppleSupport provided a large number of customer-support interactions and a high number of unique customers.

### Decision 2: A Subsample Was Used

The full dataset is very large, so a representative subset was used to keep experimentation practical.

### Decision 3: Nine Support Intents Were Used

The taxonomy was kept small enough for evaluation while covering the major issue types observed in the data.

### Decision 4: TF-IDF Was Used as the Initial Retrieval Method

It is simple, fast, reproducible, and provides a useful baseline.

### Decision 5: Top Three Historical Cases Were Retrieved

Multiple cases provide more evidence than relying on only one retrieved example.

### Decision 6: Payment Issues Are Escalated

Payment problems can require account-level investigation and should not be automatically handled by a lightweight prototype.

### Decision 7: Apple ID and Security Issues Are Escalated

Account and security issues have higher risk and require human review.

### Decision 8: Unclear Issues Are Escalated

The system should not confidently answer when there is insufficient information.

### Decision 9: Evidence Quality Is Explicitly Measured

Similarity alone is not treated as proof of correctness.

### Decision 10: Historical Responses Are Cleaned

Handles and URLs are removed from retrieved replies to make the response cleaner.

### Decision 11: A Golden Set of 200 Examples Was Used

This is within the requested 150–250 example evaluation range.

### Decision 12: Two Baselines Were Evaluated

Both a trivial baseline and a simple retrieval baseline were used for comparison.

## 12. Repository Contents

```text
apple_support_agent.ipynb
golden_eval_200.csv
requirements.txt
README.md
```
## 13. How to Run

1. Install the required packages:

```bash
pip install -r requirements.txt
Download the Customer Support on Twitter dataset.
Place twcs.csv in the project directory.
Open apple_support_agent.ipynb.
Run the notebook from top to bottom.
]
