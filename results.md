# PatchTrack Reproducibility Audit

## Research Question

**Q:** To what extent can the PatchTrack classification results (the distribution of Patch Applied / Patch Not Applied / No Patch Exists PRs, and the median integration rate) be reproduced from the authors' replication package, and how robust are they under an independent audit?

---

## Background

The report investigates the usage of ChatGPT-assisted pull requests across three integration categories:

- **Patch Applied (PA)**
- **Patch Not Applied (PN)**
- **No Patch Exists (NE)**

## Original Paper Results

The paper investigated 285 pull requests and reported the following distribution:

| Category | Count | Percentage |
|----------|-------|------------|
| Patch Applied (PA) | 116 | 40.7% |
| Patch Not Applied (PN) | 63 | 22.1% |
| No Patch Exists (NE) | 106 | 37.2% |

## Our Bounded Investigation

We ran PatchTrack on the first 80 pull requests (a 28% sample) in the paper's dataset. Our results:

| Category | Count | Percentage |
|----------|-------|------------|
| Patch Applied (PA) | 42 | 52.5% |
| No Patch Exists (NE) | 26 | 32.5% |
| Patch Not Applied (PN) | 12 | 15.0% |

---

## Adoption Rate Metric

The paper uses Jaccard's containment ratio to quantify the overlap between tokens in the ChatGPT-suggested patch (T_x) and the final integrated code (T_y).

Here, T_x and T_y are "token streams from ChatGPT and GitHub patches, respectively."

| Statistic | Paper | Our Sample |
|-----------|-------|------------|
| Median adoption rate | 25% | 48.52% |
| Third quartile | 38% | 61% |

---

## Conclusions

When comparing our bounded results to the results achieved in the paper, we observe the following.

### 1. Similar patch classification results

We observed an 11.8% increase in PA, a 4.7% decrease in NE, and a 7.1% decrease in PN.

For testing, we used a sample size of 80 out of the original 285. We chose a sample of this size because it was large enough to give us information on the classification distribution and PA adoption rate, whilst staying small enough for us to run PatchTrack.

### 2. Median patch adoption rate varies significantly

When we ran PatchTrack, we got a median adoption rate of 48.5% with a third quartile of 61%. This is nearly double the paper's median adoption rate of 25% and a 23% increase for the third quartile.

We feel that this might have occurred due to sample bias. As stated earlier, we only had a sample of 80 out of 285, roughly 28% of the original sample. It may also have occurred due to a different population. If the paper's 25% consisted of repos where the code wasn't adopted, then the median would be pulled down.

From this we can conclude that our median adoption rate was **not reproduced**.