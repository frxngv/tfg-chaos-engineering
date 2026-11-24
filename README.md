# Local Chaos Engineering Feasibility Study

This repository documents the preparation and feasibility study for a final-year
project on resilience testing of microservices in a local Kubernetes environment.

The immediate goal is to learn the tools, run small reproducible experiments,
and use the evidence to agree on a manageable project scope with the supervisors.
The final benchmark application and experimental scope have not been selected yet.

## Current stage

We are starting with containers before introducing Kubernetes, monitoring, or
fault injection. The small application in the first lesson is a learning exercise;
the final resilience study will use an existing microservices application.

## Contents

- [Lesson 01: Python application and Docker](lessons/01-containers/README.md)
- [Feasibility study log](docs/study-log.md)

## Working method

1. Explain the concept and the purpose of the experiment.
2. Read the code or configuration before running it.
3. Execute one step at a time and inspect its output.
4. Record what was actually tested, including failures and limitations.
5. Decide the next step using the results.

Documentation is written in English. Expected behavior is kept separate from
observed results. A successful deployment alone is not evidence of resilience.
