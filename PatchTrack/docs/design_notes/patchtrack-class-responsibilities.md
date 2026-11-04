# PatchTrack Class Responsibilities

## How we observed the God Class concern

`PatchTrack` in [`analyzer/main.py`](../../analyzer/main.py) was identified as a
potential God class by examining its public workflow and its methods. The
concern is not simply its size: the class coordinates several different kinds
of work, each with different reasons to change.

The class is reasonably described as a God class because a single object owns
state and methods for preparing input, interacting with external data,
managing files, classifying patches, and presenting results. This is a design
observation, not a claim that the class is necessarily broken.

## Responsibilities it encompasses

- **Configuration and state** - stores tokens, paths, PR identifiers,
  classification results, cached DataFrames, and logging configuration.
- **Input preparation and filtering** - reads project JSON, identifies merged
  pull requests, and filters projects and PRs using GitHub commit and review
  data (`prepare_data`, `_get_projects`, `_filter_projects`).
- **External data retrieval and file storage** - fetches ChatGPT code and
  GitHub PR patches, then writes them into the local directory structure
  (`_fetch_chatgpt_data`, `_fetch_github_data`).
- **Filesystem discovery and file processing** - discovers PR/project pairs,
  reads file content, and compares or processes patch and source files
  (`build_pr_project_pairs`, `read_file`, `_process_patch_pair`).
- **Classification and aggregation** - coordinates per-file comparisons,
  classifies PRs, and stores the resulting classifications (`classify`).
- **Result construction and presentation** - creates DataFrames, prints
  results, and generates visualizations (`create_dataframes`, `print_results`,
  `visualize_results`).
- **Pipeline orchestration and persistence** - runs classification and
  visualization and serializes results to disk (`run_classification`,
  `classify`).

## Why we chose not to refactor now

Recognizing a God class does not automatically make a refactor the best next
change. The current task was to understand and document the design, not to
change runtime behavior. Splitting this class would affect workflows that
cross external API calls, filesystem layouts, classification state, and
reporting; changing those boundaries without a concrete feature or defect to
address could add regression risk and review cost without a demonstrated
immediate benefit.

We therefore chose to leave the implementation unchanged for now. This is a
deferral, not a conclusion that the design is ideal. A refactor would be
worthwhile when a concrete change repeatedly requires unrelated edits, a
responsibility is difficult to test independently, or a defect points to a
specific boundary. In that case, we should extract one responsibility at a
time and verify that existing outputs and workflows remain unchanged.
