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

Use dynamic programming to find each minimum-energy seam, then remove it and repeat. With zero-based indices, let `E[r][c]` be the pixel energy and `D[r][c]` the minimum total energy of a seam from the top row to `(r, c)`.

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

The brute-force baseline recursively explores every valid seam from every top-row pixel, accumulating energy along each path and retaining the best complete seam. It does not reuse subproblem results, making it a simple reference for checking dynamic programming on small images.

### Baseline Pseudocode

The baseline uses the same energy grid and tie rule as dynamic programming. Its nested helper shares `best_energy`, `best_seam`, and `current_seam` with the enclosing function.

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

For full width reduction, replace `FIND_MINIMUM_SEAM` in `REDUCE_WIDTH` with `FIND_MINIMUM_SEAM_BRUTE_FORCE`. Exhaustive enumeration and the shared tie rule ensure both methods return the same seam at each step.

## 4. Complexity Analysis

### Assumptions and Notation

The image has `n` rows and an original width of `m` columns. Let `w` be the current width and `k = m - target_width` be the number of seams removed. Assume energy arithmetic and comparisons take constant time, and computing one pixel's energy takes constant time. Computing the energy grid then takes `Theta(nw)` time and space.

### Dynamic Programming

The `nw` table entries each check at most three predecessors. Initializing the first row and finding the bottom-row minimum take `Theta(w)` time each; reconstructing the seam takes `Theta(n)`. Finding one seam therefore takes **`Theta(nw)` time** and **`Theta(nw)` auxiliary space** for the tables and seam.

Computing energies and copying the image to remove the seam also take `Theta(nw)` time, so one complete removal still takes `Theta(nw)` time. For `k >= 1`, the widths processed are `m, m - 1, ..., target_width + 1`. The total time is

`Theta(n * k * (m + target_width + 1) / 2) = Theta(nmk)`.

Peak auxiliary space is **`Theta(nm)`**, since earlier tables and images can be released between iterations. If `k = 0`, the pseudocode returns the original image in `Theta(1)` time and auxiliary space.

### Brute Force

The baseline starts from each of the `w` top-row pixels and explores at most three choices for each of the remaining `n - 1` rows. There are at most `w * 3^(n - 1)` complete seams. Accumulating energy takes constant time per recursive call, but copying a best seam or comparing tied seams can take `O(n)` time per complete seam. Thus finding one seam has an upper bound of **`O(nw * 3^(n - 1))` time** for the pseudocode in Section 3.

This is an upper bound, since image boundaries reduce branching. A one-column image has only one seam and takes `Theta(n)` time; a one-row image takes `Theta(w)` time. For any fixed width of at least two, the number of seams grows exponentially with the number of rows.

The recursion stack, current seam, best seam, and temporary tie comparisons use **`O(n)` auxiliary space**, excluding the supplied energy grid. Including the energy grid and the copied image, one complete removal uses **`O(nw)` auxiliary space**.

For `k >= 1`, repeating this procedure gives a total-time upper bound of **`O(nmk * 3^(n - 1))`** and peak auxiliary space of **`O(nm)`**. Removing no seams takes `Theta(1)` time and auxiliary space, just as in the dynamic programming version.
