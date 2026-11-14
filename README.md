# Project 4: Image Processing

This project explores **seam carving**, a content-aware image resizing technique, through an image processing pipeline for a deep-space probe. The probe has transmitted high-resolution images of an exoplanet, but meteoroid impacts have damaged its sensor and introduced vertical bands of static. Our task is to design an algorithm that reduces image width while preserving important visual features.

An image is represented as a grid of **n rows and m columns**. Each pixel receives an energy value representing its visual importance. A vertical seam is a connected path containing one pixel in every row; each step moves down-left, straight down, or down-right. Removing a seam reduces the image width by one pixel. The algorithm removes one minimum-energy seam at a time, recalculating energies after each removal until the target width is reached. Each seam is optimal for the current image; this does not guarantee a globally optimal sequence of removals or directly detect static.

See [planning.md](planning.md) for the dynamic programming strategy, brute-force baseline, correctness argument, and planned complexity analysis. The rendered version is [planning.html](planning.html).
