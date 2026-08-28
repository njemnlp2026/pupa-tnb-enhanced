# PUPA TNB Enhanced Dataset

## Overview
This repository contains an enhanced version of the PUPA TNB dataset (https://github.com/Columbia-NLP-Lab/PAPILLON), which was first introduced in `PAPILLON: Privacy Preservation from Internet-based and Local Language Model Ensembles` (Siyan et al., 2025) https://aclanthology.org/2025.naacl-long.173/. Upon reviewing PUPA-TNB, we surfaced several issues that could compromise benchmarks:

- **28 Presidio placeholders** in user queries with missing PII units, creating disparities with their corresponding target responses
- **25 records** with incorrect PII units that either referenced absent entities in the user query or provided incomplete coverage of the PIIs present
- **1 record** lacking a target response entirely

These anomalies would hinder evaluations that use both target responses and PII units.

## Key Improvements

1. **Synthetic PII replacements**: Generated synthetic replacements for 28 Presidio placeholders in user queries using Llama-3.3-70B-Instruct, ensuring that the queries contained realistic personal information (15 person names, 11 email addresses, 2 phone numbers).
2. **PII unit corrections**: Manually corrected PII unit annotations in 25 records to ensure consistency with the queries.
3. **Target response regeneration**: Regenerated target responses using GPT-4 for the missing record and for all queries that received synthetic PII enhancements. GPT-4 was used to maintain consistency with the original PUPA-TNB dataset construction methodology.

## Examples of Improvements

### 1. Synthetic Replacements for Presidio Placeholders

| # | User Query | Target Response | PII Units |
|---|-----------|-----------------|-----------|
| 1 | **Original:** Please write an email copy saying thanks for **\<PRESIDIO_ANONYMIZED_PERSON\>** for attending the meeting and also to let... <br><br> **Improved:** Please write an email copy saying thanks for **Michael Johnson** for attending the meeting and also to let... | **Original:** Subject: Thank You for Attending the Meeting! Meeting Notes Attached Dear **\<PRESIDIO_ANONYMIZED_PERSON\>** I... <br><br> **Improved:** Subject: Thank You for Attending. Hi **Michael**, I... | **Original:** **\<presidio_anonymized_person\>** <br><br> **Improved:** **michael johnson** |
| 2 | **Original:** ...All the best, **\<PRESIDIO_ANONYMIZED_PERSON\>** <br><br> **Improved:** ...All the best, **Fatima Khan** | **Original:** ... Warm regards, **\<PRESIDIO_ANONYMIZED_PERSON\>** <br><br> **Improved:** ... Best regards, **Fatima Khan** | **Original:** **\<presidio_anonymized_person\>** \|\| ahmad <br><br> **Improved:** **fatima khan** \|\| ahmad |
| 3 | **Original:** Make my resume fit cybersecurity positions... Felipe Lozano (402) 617-4647 \| **\<PRESIDIO_ANONYMIZED_EMAIL_ADDRESS\>** \| Crete, NE ... <br><br> **Improved:** Make my resume fit cybersecurity positions... Felipe Lozano (402) 617-4647 \| **felipe.lozano@gmail.com** \| Crete, NE ... | **Original:** **Felipe Lozano** (402) 617-4647 \| **\<PRESIDIO_ANONYMIZED_EMAIL_ADDRESS\>** \| Crete, NE ... <br><br> **Improved:** ...**Felipe Lozano** (402) 617-4647 \| **felipe.lozano@gmail.com** \| Crete, NE ... | **Original:** baseballcloud \|\| lincoln \|\| lake mary \|\| bogotá \|\| take2 \|\| foundever \|\| crete \|\| hudl \|\| felipe lozano \|\| vienna \|\| doane <br><br> **Improved:** baseballcloud \|\| lincoln \|\| lake mary \|\| bogotá \|\| take2 \|\| foundever \|\| crete \|\| hudl \|\| felipe lozano \|\| vienna \|\| doane **\|\| felipe.lozano@gmail.com** |
| ... | ... | ... | ... |

Synthetic PII replacements were created for 15 person names, 11 email addresses and 2 phone numbers, totalling 28 placeholders.

### 2. PII Units Enhancements

| # | User Query | PII Units | Rationale |
|---|-----------|-----------|-----------|
| 1 | Write an email template to russian ministry of education about the invitation issuance that I have awarded scholarship in HSE university and I thought its good to start my scholarship with prep year, as I'm living in | **Original:** [location not specified in the original text] \|\| russian ministry of education \|\| hse university <br><br> **Improved:** russian ministry of education \|\| hse university | "[location not specified]" does not appear in the User Query and is therefore not a valid PII extraction. |
| 2 | If Jim puts in an order to purchase 30 shares of company A for $1.30 while the current price is $1.35, and the order is canceled immediately, what type of order did he put in? choose from the following: Limit Order Market Order Fill or Kill Order Stop-Loss Order | **Original:** jim \|\| **company a** <br><br> **Improved:** jim | "company A" is a generic placeholder used to illustrate a hypothetical scenario. It does not refer to any specific real-world entities and does not qualify as a PII unit. |
| 3 | list all job offering from SPARC companies with a URL link | **Original:** **sparc** <br><br> **Improved:** *(empty)* | "SPARC" is likely intended as SPAC (Special Purpose Acquisition Company), a type of shell corporation. It describes a classification of companies rather than a specific named entity. |
| 4 | How do I succeed financially in Syria despite all the bad conditions here? | **Original:** **syri** <br><br> **Improved:** syria | The original PII unit "syri" appears to be an incorrect truncation of the actual PII unit "syria". |
| 5 | write 5 innovative headings of 100mi - Are you ready to join hands with a revolutionary platform that combines ... | **Original:** **mi** <br><br> **Improved:** 100mi | The original PII unit "mi" is incomplete as it is missing the prefix of "100". The correct PII unit should be "100mi". |
| ... | ... | ... | ... |

25 records had their PII unit annotations corrected to ensure consistency with the queries.

### 3. Missing Target Response Replacement

| User Query | Target Response | Rationale |
|-----------|-----------------|-----------|
| Correctly format this list of words: Turtle, Garfield, Alligator, Headphones, Wedding Dress, ... | **Original:** #NAME <br><br> **Improved:** Here's the correctly formatted list of words in alphabetical order: - Alligator - America - Angle - Ant - Apple ... | "#NAME" appears to be a common spreadsheet software error code that could have replaced the correct target response value at some point during the process where the original dataset was created. |

1 record had its target response regenerated.

## Dataset Format
The dataset is provided as a single CSV file (`PUPA_TNB_enhance.csv`) containing 597 records with the following columns:
- `conversation_hash`: Unique hash identifier for the conversation
- `predicted_category`: Category of the query (e.g., "job, visa, and other applications")
- `user_query`: The user's original query text
- `target_response`: The expected model response
- `pii_units`: PII entities extracted from the query, separated by `||`
- `redacted_query`: The query with PII replaced by `[REDACTED]` placeholders
- `query_id`: Unique identifier for the query
