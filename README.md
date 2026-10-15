# Automotive Requirement-to-Test-Case Generation

A course-project proof of concept for generating structured automotive test-case drafts from natural-language requirements and a supplied feature specification.

**Course:** CSI-4130/5130: Artificial Intelligence  
**Status:** Specification and planning stage; no model, training dataset, or experimental results yet.  
**Last updated:** 2026-10-15

## Problem statement

Writing test cases from automotive requirements requires translating conditions, thresholds, timing, and expected behavior into reproducible steps. A generative model may help draft these tests, but it can omit boundary conditions, change numeric values, or invent signals and behavior.

This project investigates whether a pretrained generative model can produce useful test-case drafts for a small, fictional Automatic Emergency Braking (AEB) feature. The goal is an educational proof of concept with explicit limitations and measurable results. Generated tests require human review.

## Scope

The reference controller includes vehicle speed, target detection, TTC validity, activation/release thresholds, INCA-style input bypasses, and a fixed deceleration request. See [AEB_Spec.md](AEB_Spec.md) for the signal dictionary and requirements.

The project generates text. It does not operate a vehicle, integrate with INCA, execute HIL tests, or demonstrate collision avoidance. Perception, sensor fusion, brake hardware, diagnostics, and vehicle-program applicability are excluded. All feature definitions and calibration values are fictional.

## Proposed method

1. Review the AEB specification and prepare 10–15 pilot requirement–test-case pairs. These are initial examples, not a claim of sufficient training data.
2. Define a consistent input/output format and a review rubric.
3. Select a pretrained generative model that can be evaluated in Google Colab with a T4 GPU.
4. Establish a baseline using requirements and the relevant specification context.
5. Expand the dataset and investigate fine-tuning if data quality, time, and available resources permit. Model choice, training method, and configuration remain pending.
6. Compare the baseline with the selected improvement on held-out requirements using the same review criteria.

The input will contain a requirement ID, requirement text, and the necessary signal, calibration, bypass, and state context. The output will contain exactly three test-case sections:

- **Preconditions**
- **Test Procedure/Test Steps**
- **Acceptance Criteria**

Requirement traceability and specification version will be retained as dataset metadata. Tests will use explicit values, units, measurable outputs, and timing; missing source information should be identified rather than invented.

## Data sources and preparation

The primary source is the project's original, fictional [AEB specification](AEB_Spec.md). Requirement–test-case pairs will be authored and manually reviewed against it. AI-assisted examples will be identified as such and reviewed before inclusion. No company-confidential specifications, production signal lists, or proprietary test cases are planned for this public repository.

The initial specification has seven functional requirements; multiple pilot cases may address the same requirement. Several cases from one requirement are not independent evidence of generalization. Before claiming held-out performance, the dataset must contain enough distinct requirement groups to support a meaningful split.

Training/development and evaluation examples will be separated by requirement or closely related behavior family. Paraphrases and simple numeric variants of one requirement will remain in the same split. Evaluation answers will not be supplied as prompt examples or training targets. The common feature specification may be supplied as context, with that choice documented.

The planned dataset format is JSONL. The dataset, split sizes, and detailed schema have not yet been created.

## Evaluation plan

| Criterion | Review question |
| --- | --- |
| Requirement coverage | Does the test verify the linked conditions and expected behavior? |
| Numerical accuracy | Are values, comparison boundaries, units, and timing preserved? |
| Unsupported content | Does the model invent signals, calibrations, or behavior? |
| Testability | Are the setup and stimulus executable using the defined interface? |
| Output structure | Are the three required sections present and usable? |
| State handling | Is the initial state established correctly for hysteresis tests? |
| Missing information | Does the output identify needed information rather than fabricate it? |

A scoring rubric will be fixed before final evaluation. The report will include quantitative scores with sample counts and qualitative examples of successes and failures. Formatting checks can be automated; semantic correctness requires review against the specification. No accuracy, speed, or training improvement is claimed at this stage.

## Planned environment

- Google Colab with a T4 GPU, subject to runtime availability.
- Python for dataset handling, inference, and evaluation.
- Model, dependencies, versions, and optional fine-tuning approach: to be selected after pilot review.

There is no runnable notebook or installation procedure yet. Reproducible setup and execution instructions will be added with the implementation, including the model identifier, dependency versions, random seeds, dataset split, and training/inference settings where applicable.

## Current files

| File | Purpose |
| --- | --- |
| [README.md](README.md) | Problem statement, proposed method, data sources, and project status |
| [AEB_Spec.md](AEB_Spec.md) | CFTS-style fictional AEB specification, version 0.1 |

Planned additions include a JSONL dataset, Colab notebook, evaluation results, and final report. These are not present yet.

## Milestones and course deliverables

Dates below are transcribed from the supplied course brief; follow the course's deadline time zone.

| Milestone | Deadline / status |
| --- | --- |
| Initial specification and README | Prepared 2026-10-15 |
| Create public GitHub repository and submit its link on Moodle | October 27, 2026, 11:00 PM; pending |
| Pilot dataset and review rubric | Next development milestone |
| Baseline, selected improvement, and held-out evaluation | Planned |
| Final report PDF and final repository state | November 30, 2026, 11:00 PM |
| In-class presentations | December 1 and December 3, 2026 |

The course grades GitHub progress (10%), the final report (30%), and the presentation/demo (60%). Meaningful regular commits will record actual progress. The final report will include an abstract, introduction, related work, data, methods, experiments/results, conclusion, and citations/acknowledgements. A prerecorded presentation link and complete reproduction instructions will be added to the repository for submission.

## Limitations

This is a small educational study of a simplified fictional controller. Dataset variety, model capacity, compute availability, and review effort will limit the results. Good performance on this specification would not establish production readiness or generalization to other automotive features. Human review may also introduce subjective scoring; the rubric and examples will make judgments explicit.

## Citations and acknowledgements

- **Course brief:** *CSI-4130/5130: Artificial Intelligence — Course Project; Overview: AI in the Era of Generative Models*, supplied as `Course_proj.pdf`. Used for project deliverables, dates, and grading expectations. The course handout is not included in these initial project files.
- **Project specification:** *CFTS-AEB-001: Simplified AEB Feature Specification*, version 0.1, 2026-10-15. Original fictional project material documented in `AEB_Spec.md`.
- **AI assistance:** ChatGPT assisted with project scoping and drafting this README and the AEB specification. Any subsequent AI-assisted dataset, code, or report work will be documented and reviewed by the project author.
- **Models, software, and related research:** No model or implementation library has been selected. Add exact model cards, papers, software versions, and license information as they are used.

## Immediate next step

Review the specification and create the first 10–15 requirement–test-case pairs with traceability to requirement IDs. Use this pilot to finalize the dataset format and review rubric before selecting and running the model.
