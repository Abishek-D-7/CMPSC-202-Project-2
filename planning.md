# Project 4: Image Processing — Planning and Analysis

**Team members:** Lynsey Arthur

## 1. Problem Formulation

### The Question

The objective to Problem 4: Image Processing is to take in an image from a deep-space probe and reduce its size by removing as much static or pixels providing low-energy from it as efficiently as possible. We are given an image that is an $n * m$ grid of pixels. We must reduce the width by finding vertical "seams" of static to safely remove while keeping the integrity and quality of the photo. 

### The Objective

Our goal is to design an algorithm that correctly and quickly identifies the sequence of low-energy pixels to remove while minimizing the loss of important visual features. 

### Inputs, Outputs, and Constraints

Input: Image built of a $n * m$ grid of pixels
Output: Image with width reduced from removed static

Constraints: Must find low-energy pixels using vertical "seams" moving only down-left, straight down, or down right to adjacent columns.


## 2. Algorithmic Strategy and Pseudocode


### Strategy: Dynamic Programming

Use dynamic programming to find a minimum-energy vertical seam in the current image. A seam contains exactly one pixel in each row, and consecutive pixels can differ by at most one column. Its total energy is the sum of its pixels' energy values. After finding a seam, remove it, update the image's energy values, and repeat until the desired width is reached.

Let `E[r][c]` be the energy of the pixel in row `r`, column `c`, using zero-based indices. The energy calculation is provided by the image-processing pipeline; larger values mean greater visual importance. Define `D[r][c]` as the minimum total energy of any seam starting in the top row and ending at `(r, c)`.

- **Base case:** `D[0][c] = E[0][c]` for every column.
- **Transition:** `D[r][c] = E[r][c] + min(D[r - 1][p])`, where `p` is a valid column among `c - 1`, `c`, and `c + 1`.
- **Boundary rule:** Ignore predecessors outside the image.
- **Reconstruction:** Store the predecessor column chosen for each state. Start at the minimum entry in the bottom row and follow predecessors upward to recover the seam.
- **Tie rule:** If multiple choices have the same energy, choose the smallest column index for reproducible results.

### Pseudocode

```text
FIND_MINIMUM_SEAM(E):
    n = number of rows in E
    w = number of columns in E
    require n >= 1 and w >= 1
    create tables D[n][w] and parent[n][w]

    for c = 0 to w - 1:
        D[0][c] = E[0][c]
        parent[0][c] = NONE

    for r = 1 to n - 1:
        for c = 0 to w - 1:
            best = max(0, c - 1)
            for p = max(0, c - 1) to min(w - 1, c + 1):
                if D[r - 1][p] < D[r - 1][best]:
                    best = p
            D[r][c] = E[r][c] + D[r - 1][best]
            parent[r][c] = best

    end = 0
    for c = 1 to w - 1:
        if D[n - 1][c] < D[n - 1][end]:
            end = c

    create array seam[n]
    seam[n - 1] = end
    for r = n - 1 down to 1:
        seam[r - 1] = parent[r][seam[r]]
    return seam

REDUCE_WIDTH(image, target_width):
    n = number of rows in image
    w = number of columns in image
    require n >= 1 and 1 <= target_width <= w

    while w > target_width:
        E = COMPUTE_ENERGIES(image)
        seam = FIND_MINIMUM_SEAM(E)
        create new_image with n rows and w - 1 columns

        for r = 0 to n - 1:
            destination = 0
            for c = 0 to w - 1:
                if c != seam[r]:
                    new_image[r][destination] = image[r][c]
                    destination = destination + 1

        image = new_image
        w = w - 1

    return image
```

`COMPUTE_ENERGIES` applies the pipeline's pixel-energy rule to the current image. Recomputing after each removal accounts for the changed pixel neighborhoods.

### Correctness Argument

We prove by induction on the row that `D[r][c]` is the minimum energy of any valid seam ending at `(r, c)`. In row zero, the only such path contains that pixel alone, so its energy is `E[0][c]`. For a later row, every valid seam reaching `(r, c)` must come from one of the valid adjacent columns in the previous row. By the induction hypothesis, the table already stores the minimum cost of reaching each predecessor. Choosing the cheapest predecessor and adding `E[r][c]` therefore gives the minimum cost of reaching `(r, c)`.

Every complete vertical seam ends somewhere in the bottom row, so the minimum entry in that row is the minimum energy of any complete seam. Following the stored predecessors produces a path with one pixel per row and only allowed moves, with exactly that energy. Removing those pixels preserves the order of all remaining pixels and reduces the width by one. Repeating this step reaches the requested width.

This proves that each removed seam is optimal for the image at that step. Repeated removal makes a greedy choice between iterations; it does not guarantee the globally minimum total energy over every possible sequence of removals, because each removal changes the image and subsequent energies.

## 3. Baseline Solution

TODO: Describe a simple reference algorithm, provide pseudocode, and explain how it will be compared with the proposed solution.

## 4. Complexity Analysis

TODO: Analyze time and space for both algorithms, with mathematical justification and clearly stated assumptions.
