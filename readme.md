# Hashtag Recommendation Using Association Rule Mining

## 1. Project Overview

The goal of this project is to build a general cross-platform hashtag recommendation system using hashtag co-occurrence patterns from social media datasets.

The available datasets come from different platforms and do not all contain the same attributes. However, all useful datasets contain hashtags, and most contain basic engagement metrics such as likes, comments, and shares. Therefore, the project should focus on hashtag transactions as the core input and use engagement metrics only as an optional ranking factor.

Recommended project title:

```text
General Cross-Platform Hashtag Recommendation Using Association Rule Mining
```

The project should not be framed as a region-based recommendation system unless all datasets consistently contain reliable region information. It should also not depend on captions, saves, content type, or music because those fields are missing from some datasets.

## 2. Finalized Project Direction

The system will learn patterns such as:

```text
["python", "datascience"] -> ["machinelearning"]
```

This means that when a user provides hashtags such as `python` and `datascience`, the system may recommend `machinelearning` because it frequently appears with them in previous posts.

The project will compare three methods:

1. Frequency-Based Hashtag Recommendation
2. Apriori Association Rule Mining
3. FP-Growth with Engagement-Aware Ranking

The first method acts as a baseline. Apriori provides a classic and explainable association rule mining approach. FP-Growth provides a faster and more scalable association mining approach, while engagement-aware ranking improves the practical usefulness of recommendations.

## 3. Available Dataset Summary

### 3.1 TikTok Dataset

Observed columns:

```text
video_id
author
likes
comments
shares
plays
hashtags
music
```

Useful columns:

```text
video_id
likes
comments
shares
plays
hashtags
```

Columns to ignore for the main model:

```text
author
music
```

Reason:

The author and music fields may be useful for other types of analysis, but they are not necessary for association rule based hashtag recommendation.

### 3.2 Updated Social Dataset

Observed columns:

```text
Post_ID
Platform
Hashtag
Content_Type
Region
Views
Likes
Shares
Comments
Engagement_Level
```

Useful columns:

```text
Post_ID
Platform
Hashtag
Views
Likes
Shares
Comments
Engagement_Level
```

Optional columns:

```text
Content_Type
Region
```

Reason:

Region and content type can be used for extra exploratory analysis, but they should not be required for the main model because other datasets do not consistently contain them.

### 3.3 Instagram Dataset

Observed columns:

```text
Impressions
From Home
From Hashtags
From Explore
From Other
Saves
Comments
Shares
Likes
Profile Visits
Follows
Caption
Hashtags
```

Useful columns:

```text
Impressions
Saves
Comments
Shares
Likes
Hashtags
```

Optional column:

```text
Caption
```

Columns to ignore for the main model:

```text
From Home
From Hashtags
From Explore
From Other
Profile Visits
Follows
```

Reason:

These columns are platform-specific Instagram analytics. They can be used for separate Instagram performance analysis, but they should not be part of the common cross-platform hashtag recommendation model.

## 4. Final Unified Dataset Structure

All datasets should be converted into one common structure before modeling.

Recommended final columns:

```text
post_id
platform
hashtags
likes
comments
shares
views_or_plays_or_impressions
saves
engagement_level
engagement_level_numeric
engagement_score
engagement_rate
```

Required columns:

```text
post_id
platform
hashtags
likes
comments
shares
```

Optional columns:

```text
views_or_plays_or_impressions
saves
engagement_level
engagement_level_numeric
engagement_rate
```

The model can still be trained even if some optional columns are missing.

## 5. Dataset Mapping

### 5.1 TikTok Dataset Mapping

```text
video_id -> post_id
"TikTok" -> platform
hashtags -> hashtags
likes -> likes
comments -> comments
shares -> shares
plays -> views_or_plays_or_impressions
blank -> saves
blank -> engagement_level
```

### 5.2 Updated Social Dataset Mapping

```text
Post_ID -> post_id
Platform -> platform
Hashtag -> hashtags
Likes -> likes
Comments -> comments
Shares -> shares
Views -> views_or_plays_or_impressions
blank -> saves
Engagement_Level -> engagement_level
```

### 5.3 Instagram Dataset Mapping

```text
generated ID -> post_id
"Instagram" -> platform
Hashtags -> hashtags
Likes -> likes
Comments -> comments
Shares -> shares
Impressions -> views_or_plays_or_impressions
Saves -> saves
blank -> engagement_level
```

## 6. Hashtag Cleaning and Preprocessing

Hashtag cleaning is one of the most important parts of this project. Poor hashtag cleaning will produce weak association rules.

### 6.1 Required Cleaning Steps

1. Convert all hashtags to lowercase.
2. Remove the `#` symbol.
3. Remove extra spaces.
4. Split hashtags correctly.
5. Remove duplicate hashtags within the same post.
6. Remove rows with missing hashtags.
7. Remove rows with only one hashtag.
8. Fix encoding issues such as `Â`.
9. Remove empty strings after splitting.
10. Standardize similar tags where obvious.

Example:

```text
"#DataScience #Python #AI"
```

Should become:

```text
["datascience", "python", "ai"]
```

Example from TikTok:

```text
"fyp, publicinterview, rizz"
```

Should become:

```text
["fyp", "publicinterview", "rizz"]
```

### 6.2 Handling Different Hashtag Formats

Different datasets may store hashtags differently:

```text
#python #datascience #ai
python, datascience, ai
fyp, viral, dance
```

The preprocessing function should support both space-separated and comma-separated hashtags.

### 6.3 Rows to Remove

Remove:

```text
rows with no hashtags
rows with only one hashtag
duplicate rows
rows with invalid engagement values
```

Rows with only one hashtag are not useful for association rule mining because association rules require relationships between items.

## 7. Engagement Score Strategy

Engagement should not be mandatory for training. It should be used only to rank or improve recommendations where available.

### 7.1 Basic Engagement Score

The simple version is:

```text
engagement_score = likes + comments + shares
```

If saves are available:

```text
engagement_score = likes + comments + shares + saves
```

### 7.2 Recommended Weighted Engagement Score

The recommended version is:

```text
engagement_score = likes + (2 * comments) + (3 * shares) + (2 * saves)
```

If saves are missing:

```text
engagement_score = likes + (2 * comments) + (3 * shares)
```

Reason:

Likes are easy to give, while comments and shares usually show stronger user interest. Shares are especially important because they spread the post to more users.

### 7.3 Engagement Rate

If views, plays, or impressions are available:

```text
engagement_rate = engagement_score / views_or_plays_or_impressions
```

If views, plays, or impressions are missing:

```text
engagement_rate = blank / NaN
```

Do not invent fake values for missing engagement rate.

### 7.4 Engagement Level Conversion

For datasets with engagement level:

```text
Low = 1
Medium = 2
High = 3
```

This can be stored as:

```text
engagement_level_numeric
```

This field should be used only for supporting analysis, not as the main model input.

## 8. Transaction Dataset for Association Rule Mining

Association rule mining does not use the raw table directly. It requires transactions.

Each post becomes one transaction:

```text
Post 1: ["challenge", "photochallenge", "challengeaccepted"]
Post 2: ["education", "studentlife", "studytips", "stem"]
Post 3: ["python", "datascience", "machinelearning", "ai"]
```

The transaction dataset is then converted into one-hot encoded format.

Example:

| post_id | python | datascience | ai | fitness |
|---|---:|---:|---:|---:|
| Post_1 | 1 | 1 | 1 | 0 |
| Post_2 | 0 | 0 | 0 | 1 |

This one-hot encoded format is used by Apriori and FP-Growth.

## 9. Algorithm 1: Frequency-Based Hashtag Recommendation

### 9.1 Purpose

This is the baseline model. It recommends hashtags based on global frequency or co-occurrence frequency.

### 9.2 Implementation Approach

There are two possible frequency-based methods:

Global frequency:

```text
Recommend the most common hashtags in the dataset.
```

Co-occurrence frequency:

```text
Given an input hashtag, recommend hashtags that appear most often with it.
```

Example:

```text
Input: ["fitness"]
Recommendation: ["gym", "workout", "health"]
```

### 9.3 Suggested Implementation Steps

1. Clean all hashtags.
2. Create hashtag transactions.
3. Count individual hashtag frequency.
4. Count pairwise hashtag co-occurrence.
5. For a given input hashtag or set of hashtags, recommend the most frequently co-occurring hashtags.
6. Exclude hashtags already provided as input.

### 9.4 Validation Metrics

Use:

```text
Precision@K
Recall@K
Hit Rate@K
Coverage
Runtime
```

Recommended values of K:

```text
K = 3
K = 5
K = 10
```

### 9.5 Strengths

```text
Simple to implement
Easy to explain
Good baseline
Fast
```

### 9.6 Weaknesses

```text
May recommend overly popular hashtags
Does not capture deeper association rules
May produce generic recommendations like fyp, viral, trending
```

## 10. Algorithm 2: Apriori Association Rule Mining

### 10.1 Purpose

Apriori is a classic association rule mining algorithm. It finds frequent itemsets and generates rules based on support and confidence.

Example rule:

```text
["python", "datascience"] -> ["machinelearning"]
```

### 10.2 Implementation Approach

Use cleaned hashtag transactions and convert them into one-hot encoded format.

Then apply Apriori to find frequent itemsets.

In Google Colab, the `mlxtend` library can be used:

```python
from mlxtend.frequent_patterns import apriori, association_rules
```

### 10.3 Suggested Implementation Steps

1. Clean hashtags.
2. Convert posts into hashtag transactions.
3. One-hot encode transactions.
4. Apply Apriori.
5. Generate association rules.
6. Filter rules using support, confidence, and lift.
7. Use rules to recommend hashtags.

### 10.4 Important Parameters

Recommended starting values:

```text
min_support = 0.01
min_confidence = 0.3
min_lift = 1.0
```

If too few rules are generated, lower `min_support`.

If too many weak rules are generated, increase `min_confidence` or `min_lift`.

### 10.5 Validation Metrics

Use rule quality metrics:

```text
support
confidence
lift
leverage
conviction
number of rules generated
runtime
```

Use recommendation metrics:

```text
Precision@K
Recall@K
Hit Rate@K
Mean Reciprocal Rank
Coverage
```

### 10.6 Strengths

```text
Easy to explain
Well-known algorithm
Good for academic/report writing
Produces interpretable rules
```

### 10.7 Weaknesses

```text
Can be slow on large datasets
Needs careful support threshold tuning
May miss rare but useful hashtag combinations
```

## 11. Algorithm 3: FP-Growth with Engagement-Aware Ranking

### 11.1 Purpose

FP-Growth is a faster association rule mining algorithm than Apriori. It finds frequent itemsets without repeatedly scanning the dataset.

For this project, FP-Growth should be combined with engagement-aware ranking to make recommendations more useful.

### 11.2 Implementation Approach

Use FP-Growth to generate association rules, then rank the recommended hashtags using both rule strength and engagement score.

In Google Colab, `mlxtend` can also be used:

```python
from mlxtend.frequent_patterns import fpgrowth, association_rules
```

### 11.3 Suggested Implementation Steps

1. Clean hashtags.
2. Create transaction dataset.
3. One-hot encode transactions.
4. Apply FP-Growth.
5. Generate association rules.
6. Calculate average engagement score for hashtags or hashtag combinations.
7. Rank recommendations using rule metrics and engagement.

### 11.4 Recommended Ranking Formula

Basic rule score:

```text
rule_score = confidence * lift
```

Engagement-aware score:

```text
final_score = confidence * lift * normalized_engagement_score
```

Where:

```text
normalized_engagement_score = scaled value between 0 and 1
```

If engagement is missing:

```text
final_score = confidence * lift
```

### 11.5 Important Parameters

Recommended starting values:

```text
min_support = 0.01
min_confidence = 0.3
min_lift = 1.0
```

For large datasets:

```text
min_support = 0.005
```

For small datasets:

```text
min_support = 0.02
```

### 11.6 Validation Metrics

Use rule quality metrics:

```text
support
confidence
lift
leverage
conviction
number of rules generated
runtime
```

Use recommendation metrics:

```text
Precision@K
Recall@K
Hit Rate@K
Mean Reciprocal Rank
Coverage
```

Use engagement-based metrics:

```text
average engagement score of recommended hashtags
average engagement rate of recommended hashtags
comparison of engagement-aware vs non-engagement ranking
```

### 11.7 Strengths

```text
Faster than Apriori
Better for larger datasets
Can be improved using engagement ranking
Likely to be the best final model for this project
```

### 11.8 Weaknesses

```text
Slightly harder to explain than Apriori
Still depends on good hashtag cleaning
Engagement-aware ranking is only possible where engagement data exists
```

## 12. Validation and Evaluation Strategy

The project should evaluate both rule quality and recommendation quality.

### 12.1 Rule Quality Metrics

Support:

```text
How often an itemset appears in the dataset.
```

Confidence:

```text
How often the consequent appears when the antecedent appears.
```

Lift:

```text
How much stronger the rule is compared with random chance.
```

Lift interpretation:

```text
lift > 1: positive association
lift = 1: no meaningful association
lift < 1: negative association
```

Leverage:

```text
Difference between observed co-occurrence and expected co-occurrence.
```

Conviction:

```text
Measures implication strength of a rule.
```

### 12.2 Recommendation Quality Metrics

Use a hide-one-hashtag validation method.

Example:

Original transaction:

```text
["python", "datascience", "machinelearning", "ai"]
```

Input:

```text
["python", "datascience", "machinelearning"]
```

Hidden expected hashtag:

```text
"ai"
```

The recommendation is successful if `ai` appears in the top K recommendations.

Recommended metrics:

```text
Precision@K
Recall@K
Hit Rate@K
Mean Reciprocal Rank
Coverage
```

### 12.3 Runtime Comparison

Measure runtime for:

```text
Frequency-based method
Apriori
FP-Growth
```

Expected result:

```text
Frequency-based method should be fastest.
FP-Growth should be faster than Apriori for larger datasets.
Apriori may become slower as the number of unique hashtags increases.
```

## 13. Recommended Comparison Table

The final report should include a table like this:

| Method | Support | Confidence | Lift | Precision@5 | Recall@5 | Hit Rate@5 | Runtime | Notes |
|---|---:|---:|---:|---:|---:|---:|---:|---|
| Frequency-Based | N/A | N/A | N/A | value | value | value | value | Baseline |
| Apriori | value | value | value | value | value | value | value | Explainable |
| FP-Growth + Engagement Ranking | value | value | value | value | value | value | value | Recommended final model |

## 14. Exploratory Data Analysis

Before modeling, perform EDA to understand the datasets.

Recommended EDA:

1. Number of posts per dataset.
2. Number of posts per platform.
3. Number of unique hashtags.
4. Most frequent hashtags.
5. Average number of hashtags per post.
6. Distribution of likes, comments, and shares.
7. Top hashtags by average engagement score.
8. Top hashtags by average engagement rate.
9. Missing value analysis.
10. Duplicate post analysis.

Useful plots:

```text
bar chart of top 20 hashtags
bar chart of platform distribution
histogram of engagement score
boxplot of engagement by platform
bar chart of average engagement for top hashtags
```

## 15. Important Data Analytics Considerations

### 15.1 Avoid Overclaiming

Do not claim that the model recommends hashtags based on global geographic trends unless region data is available and reliable across all datasets.

Better wording:

```text
The model identifies general cross-platform hashtag co-occurrence patterns.
```

Avoid:

```text
The model identifies worldwide regional hashtag trends.
```

### 15.2 Platform Bias

Some platforms may have more rows than others. If TikTok has many more rows than Instagram, the model may become biased toward TikTok hashtag patterns.

Possible solution:

```text
Analyze combined dataset and platform-specific subsets separately.
```

### 15.3 Popular Hashtag Bias

Very common hashtags like `fyp`, `viral`, and `trending` may dominate recommendations.

Possible solution:

```text
Create two result sets:
1. with all hashtags
2. with overly generic hashtags removed
```

Potential generic hashtags:

```text
fyp
viral
trending
explore
reels
shorts
```

Do not automatically remove them without reporting it. Mention it as a preprocessing decision.

### 15.4 Rare Hashtag Problem

Rare hashtags may be useful but may not meet minimum support.

Possible solution:

```text
Use lower min_support values for experiments.
```

### 15.5 Engagement Bias

Posts with very high engagement may dominate engagement-aware ranking.

Possible solution:

```text
Normalize engagement scores.
Use engagement rate where views, plays, or impressions exist.
Use log transformation for very skewed engagement values.
```

Example:

```text
log_engagement_score = log(1 + engagement_score)
```

## 16. Google Colab Implementation Plan

Recommended notebook sections:

1. Import libraries.
2. Load datasets.
3. Standardize column names.
4. Merge datasets.
5. Clean hashtags.
6. Calculate engagement score.
7. Perform EDA.
8. Create transaction dataset.
9. One-hot encode hashtags.
10. Implement frequency-based recommender.
11. Implement Apriori.
12. Implement FP-Growth.
13. Add engagement-aware ranking.
14. Evaluate models.
15. Compare results.
16. Final conclusion.

Recommended libraries:

```python
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
import seaborn as sns
from mlxtend.preprocessing import TransactionEncoder
from mlxtend.frequent_patterns import apriori, fpgrowth, association_rules
```

If needed:

```python
!pip install mlxtend
```

## 17. Final Model Recommendation

The expected best final model is:

```text
FP-Growth with Engagement-Aware Ranking
```

Reason:

```text
It is faster than Apriori, suitable for larger hashtag datasets, and can rank recommendations using engagement signals where available.
```

However, the report should still include Frequency-Based Recommendation and Apriori because they provide useful comparison points.

## 18. Final Methodology Statement

The following statement can be used in the report:

```text
This project uses social media posts as transaction baskets, where each basket contains the hashtags used in a single post. After cleaning and standardizing hashtags across multiple platform datasets, association rule mining algorithms are applied to discover frequent hashtag co-occurrence patterns. A frequency-based method is used as a baseline, Apriori is used as a classic explainable association rule mining approach, and FP-Growth is used as a more efficient algorithm. Engagement metrics such as likes, comments, shares, saves, views, plays, and impressions are used as optional ranking factors where available. Since not all datasets contain the same engagement-related fields, the core recommendation model is based on hashtag co-occurrence, while engagement-aware ranking is applied only when suitable data exists.
```

## 19. Final Deliverables

Recommended final deliverables:

```text
1. Cleaned merged dataset
2. Google Colab notebook
3. EDA visualizations
4. Frequent itemsets
5. Association rules
6. Recommendation function
7. Algorithm comparison table
8. Final report or presentation
```

## 20. Conclusion

The project should focus on a clean, realistic, and explainable recommendation pipeline. The most important input is the hashtag list from each post. Engagement data should improve ranking but should not be required for model training. This approach allows all three datasets to be used together while still making proper use of engagement metrics where available.

Final selected methods:

```text
1. Frequency-Based Hashtag Recommendation
2. Apriori Association Rule Mining
3. FP-Growth with Engagement-Aware Ranking
```

Final required dataset columns:

```text
post_id
platform
hashtags
likes
comments
shares
```

Final optional dataset columns:

```text
views_or_plays_or_impressions
saves
engagement_level
engagement_score
engagement_rate
```
