# Project 4: Image Processing

This project explores **seam carving**, a content-aware image resizing technique, through an image processing pipeline for a deep-space probe. The probe has transmitted high-resolution images of an exoplanet, but meteoroid impacts have damaged its sensor and introduced vertical bands of static. Our task is to design an algorithm that reduces image width while preserving important visual features.

An image is represented as a grid of **n rows and m columns**. Each pixel receives an energy value representing its visual importance. A vertical seam is a connected path containing one pixel in every row; each step moves down-left, straight down, or down-right. Removing a seam reduces the image width by one pixel. The goal is to select a sequence of seams that minimizes the loss of high-contrast details.

## Deliverable 1: Planning and Analysis

The first deliverable focuses on the theoretical design before implementation. The team will define the problem, propose an algorithm with detailed pseudocode, develop a baseline for comparison, and justify the time and space complexity of both approaches.

The final submission will be a concise writeup in [planning.md](planning.md).

## Team Responsibilities

| Team member | Responsibility | Working file |
| --- | --- | --- |
| Person 1 | Define inputs, outputs, constraints, and the optimization objective | [Problem formulation](planning/01-problem-formulation.md) |
| Person 2 | Select a design paradigm and develop pseudocode and a correctness argument | [Algorithmic strategy](planning/02-algorithmic-strategy.md) |
| Person 3 | Design a simple baseline and identify comparison cases | [Baseline solution](planning/03-baseline-solution.md) |
| Person 4 | Analyze running time and space requirements with mathematical justification | [Complexity analysis](planning/04-complexity-analysis.md) |

## Collaboration

Each team member will draft their assigned section in the corresponding working file. The team will then review the sections together and combine them into `planning.md`, using consistent notation and assumptions throughout.

Before finalizing the analysis, agree on how pixel energy is calculated and updated after removal. Also distinguish between finding the best individual seam and finding the best overall sequence: repeatedly removing the cheapest current seam does not automatically guarantee a globally optimal sequence.

