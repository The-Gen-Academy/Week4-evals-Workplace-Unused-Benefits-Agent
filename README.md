# Unused Benefits Agent | Langfuse + Jev

Agent evaluation done for an employee-benefits decision agent. The
agent uses LangChain and an LLM to assess one employee-and-claim object
against three fictional policies (learning, wellness, internet) and eventually evaluated using the help of Langfuse and Jev.

The evaluation process is as follows: a golden dataset, Langfuse
experiments and traces, human annotation, prompt improvement, regression
checks, and Jev as an independent decision-model evaluator.

## Setup

Use Python 3.11 in VS Code with the Python and Jupyter extensions. Open
the python notebook, select your Python kernel, and run
the first `%pip install` cell. Restart the kernel if packages were
updated.

Create a `.env` file in the notebook's working directory with:

``` dotenv
OPENAI_API_KEY=...
LANGFUSE_PUBLIC_KEY=...
LANGFUSE_SECRET_KEY=...
LANGFUSE_HOST=https://us.cloud.langfuse.com
```

The host must match the region of your Langfuse project. Never commit
the `.env` file. Jev needs a separately configured provider connection
in Langfuse; LangChain does not supply Jev credentials.

## Run the notebook

1.  **Baseline:** Run the single smoke check, upload or reuse the
    18-case golden dataset, then execute the V1 experiment. Inspect the
    reported failures.
2.  **Human review:** In Langfuse, use an Annotation Queue named
    `unused-benefits-v1-human-review` with two Score Configs:
    `decision_correct` (numeric, 0 or 1) and `human_decision`
    (categorical: `eligible`, `not_eligible`, `needs_review`). The
    notebook selects few genuine failure and success cases, adds
    their traces to the existing queue, and retrieves saved human
    decisions from the v3 Scores API. Review and complete the items
    in the UI before fetching the labels.
3.  **Improve:** Run the V2 prompt experiment on the same golden
    dataset. Compare V1/V2 and run the focused regression experiment for
    known V1 failures.
4.  **Jev:** In Langfuse, configure a Jev decision-model evaluator with
    the applicable policy, employee claim, and agent output. Exclude the
    golden expected answer from its inputs. Use **Experiments → Run Evaluator** on both V1 and V2 experiments.
5.  **Calibrate:** Fetch Jev scores with the v3 Scores API and match
    each score to its case and version through the v2 Observations API
    (`experiment-item-task` metadata). Compare V2 Jev decisions against
    all 18 golden labels, and V1 Jev decisions against the ten
    human-reviewed labels.
6.  **Finish:** Run the final summary cell to report agent accuracy,
    regression results, evaluator coverage, and agreement.

The Annotation Queue must exist and have both score configurations
attached before the queue cell runs. The notebook looks it up by name.

## Results saved in the supplied notebook

  Check                                          Recorded result
  ----------------------------- --------------------------------
  V1 decision accuracy                             55.6% (10/18)
  V2 decision accuracy                              100% (18/18)
  Known V1 failures corrected                                8/8
  Human/Jev agreement             80% (8/10 reviewed V1 cases)
  Jev vs. golden labels                   100% (18/18 V2 cases)

The 100% V2 accuracy is on the development benchmark used for prompt
refinement; it is not evidence of performance on unseen claims.
Human/Jev agreement on selected cases is an initial calibration
check, not production validation.
