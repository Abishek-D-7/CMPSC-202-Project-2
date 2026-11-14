# Project 4: Image Processing — Planning and Analysis

**Team members:** Lynsey Arthur, Abishek Dhakal, Jayden, and Sam

## 1. Problem Formulation

### The Question

The objective of Problem 4: Image Processing is to reduce the width of an image transmitted by a deep-space probe while preserving important visual features. The image is an $n \times m$ grid of pixels, and sensor damage has introduced vertical bands of static. We reduce its width by removing low-energy vertical seams. The algorithm selects pixels by energy rather than directly detecting static.

### The Objective

Our goal is to efficiently find and remove a minimum-energy vertical seam at each step until the image reaches the desired width. Each pixel has an energy value representing its visual importance; higher values represent stronger contrast or important details. A seam's energy is the sum of its pixels' energies. After each removal, energies are recalculated for the updated image. Each selected seam has the lowest total energy for the current image, although this does not guarantee the globally best sequence of seam removals.

### Inputs, Outputs, and Constraints

Input: An image represented by an $n \times m$ grid of pixels, a desired width `target_width`, and a pixel-energy rule supplied by the image-processing pipeline.

Output: An image with $n$ rows and `target_width` columns, produced by removing one minimum-energy seam at a time.

Constraints: The image must have at least one row, and `1 <= target_width <= m`. Each vertical seam contains exactly one pixel in every row, from top to bottom. Consecutive pixels may move only down-left, straight down, or down-right, staying within the image boundaries. Removing a seam preserves the order of the remaining pixels in each row and reduces the width by one.

## 2. Algorithmic Strategy and Pseudocode

### Strategy: Dynamic Programming

Use dynamic programming to find a minimum-energy vertical seam in the current image. A seam contains exactly one pixel in each row, and consecutive pixels can differ by at most one column. Its total energy is the sum of its pixels' energy values. After finding a seam, remove it, update the image's energy values, and repeat until the desired width is reached.

Let `E[r][c]` be the energy of the pixel in row `r`, column `c`, using zero-based indices. The energy calculation is provided by the image-processing pipeline; larger values mean greater visual importance. Define `D[r][c]` as the minimum total energy of any seam starting in the top row and ending at `(r, c)`.

- **Base case:** `D[0][c] = E[0][c]` for every column.
- **Transition:** `D[r][c] = E[r][c] + min(D[r - 1][p])`, where `p` is a valid column among `c - 1`, `c`, and `c + 1`.
- **Boundary rule:** Ignore predecessors outside the image.
- **Reconstruction:** Store the predecessor column chosen for each state. Start at the minimum entry in the bottom row and follow predecessors upward to recover the seam.
- **Tie rule:** If multiple choices have the same energy, choose the smallest column index for reproducible results. For complete seams with equal energy, this orders their column sequences lexicographically from the bottom row upward (compare the bottom columns first, then the preceding rows).

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

### Brute-Force Approach

For our baseline solution, we will use a brute-force algorithm that checks every possible valid vertical seam in the current image. Starting from every pixel in the top row, the algorithm recursively tries every valid next step: down-left, straight down, or down-right. When a path reaches the bottom row, it calculates the total energy of that seam and compares it to the best seam found so far.

The baseline returns the valid seam with the smallest total energy. Unlike our dynamic programming approach, this algorithm does not save the results of smaller subproblems, so many similar paths are explored repeatedly. This makes it inefficient for larger images, but it gives us a simple reference algorithm that we can use to check the correctness and performance of our proposed solution.

### Baseline Pseudocode

The baseline uses the same energy grid, valid moves, and tie rule as dynamic programming. Its recursive helper shares `best_energy`, `best_seam`, and `current_seam` with the enclosing function. Energies are accumulated along each path; the current path is copied only when it becomes the best seam.

```text
FIND_MINIMUM_SEAM_BRUTE_FORCE(E):
    n = number of rows in E
    w = number of columns in E
    require n >= 1 and w >= 1
    best_energy = infinity
    best_seam = NONE
    current_seam = empty list

    // Nested helper: best_energy and best_seam are shared, mutable state.
    EXPLORE_SEAM(row, col, current_energy):
        append col to current_seam
        current_energy = current_energy + E[row][col]

        if row == n - 1:
            if best_seam == NONE or current_energy < best_energy:
                best_energy = current_energy
                best_seam = copy of current_seam
            else if current_energy == best_energy:
                if reverse(current_seam) is lexicographically smaller
                   than reverse(best_seam):
                    best_seam = copy of current_seam
        else:
            for next_col = max(0, col - 1) to min(w - 1, col + 1):
                EXPLORE_SEAM(row + 1, next_col, current_energy)

        // Backtrack after both complete and incomplete paths.
        remove the last column from current_seam

    for start_col = 0 to w - 1:
        EXPLORE_SEAM(0, start_col, 0)

    return best_seam
```

To compare full width reduction, use the same `REDUCE_WIDTH` procedure with `FIND_MINIMUM_SEAM_BRUTE_FORCE` in place of `FIND_MINIMUM_SEAM`. Both algorithms recompute energies on the same updated image after each removal. The baseline enumerates every valid seam, so selecting the least energy, with the shared tie rule, returns the same seam as dynamic programming.

## 4. Complexity Analysis

TODO: Analyze time and space for both algorithms, with mathematical justification and clearly stated assumptions.
