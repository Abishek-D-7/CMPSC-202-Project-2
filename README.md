# Project 4: Image Processing

This project explores **seam carving**, a content-aware image resizing technique, through an image processing pipeline for a deep-space probe. The probe has transmitted high-resolution images of an exoplanet, but meteoroid impacts have damaged its sensor and introduced vertical bands of static. Our task is to design an algorithm that reduces image width while preserving important visual features.

An image is represented as a grid of **n rows and m columns**. Each pixel receives an energy value representing its visual importance. A vertical seam is a connected path containing one pixel in every row; each step moves down-left, straight down, or down-right. Removing a seam reduces the image width by one pixel. The goal is to select a sequence of seams that minimizes the loss of high-contrast details.
