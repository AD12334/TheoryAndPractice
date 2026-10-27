
## Chart Data

| Measurement | Last Week (First 80 PRs) | Week 8 (Last 80 PRs) |
|---|---:|---:|
| Median adoption | 48.52% | 33% |
| Third quartile (Q3) | 61% | 65% |

## Adoption rates

| Classification | Week 7 | Week 8 | Change |
|---|---:|---:|---:|
| PA | 42 (52.5%) | 36 (45%) | -6 patches (-7.5 pp) |
| NE | 26 (32.5%) | 34 (42.5%) | +8 patches (+10 pp) |
| PN | 12 (15%) | 10 (12.5%) | -2 patches (-2.5 pp) |
| **Total** | **80 (100%)** | **80 (100%)** | **0 patches** |

### Key Findings
- PA decreased by 7.5 percentage points.
- NE increased by 10 percentage points.
- PN decreased by 2.5 percentage points.


# Conclusions

When comparing our bounded results to the results achieved in the previous investigation, we observe the following.

## Median Patch Adoption Rate Decreases

When we ran PatchTrack on the last 80 samples of the data, we got a median adoption rate of 33%, compared to our previous median adoption rate of 48.5%. This is a decrease of 15.5 percentage points. We feel that this might have occurred due to a difference in the sample of pull requests. As stated earlier, we used the last 80 samples for this investigation instead of the first 80. This suggests that the median adoption rate can vary depending on which pull requests are examined.

## Third Quartile Patch Adoption Rate Increases

When we ran PatchTrack, we got a third quartile adoption rate of 65%, compared to our previous third quartile of 61%. This is an increase of 4 percentage points. This shows that although the median adoption rate decreased, the upper quartile increased. This suggests that some pull requests in the new sample had higher adoption rates, despite the overall median being lower.

## Patch Adoption Results Vary Between Samples

When comparing the first 80 samples to the last 80 samples, we observed a significant difference in the median adoption rate, which went from 48.5% to 33%. This suggests that the results may be influenced by the selection of pull requests. From this, we can conclude that the patch adoption rate was not consistent across the two samples. Further investigation using a larger sample size would be needed to determine whether the original paper's findings can be reproduced across different sets of pull requests.