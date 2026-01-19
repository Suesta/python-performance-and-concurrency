# python-performance-and-concurrency

Algorithm performance and concurrency in Python: complexity analysis, profiling, efficient CSV processing with pandas, multiprocessing, multithreading and concurrent image downloads from a public API.

---

## 1. Project overview

This repository contains a single Jupyter Notebook developed as part of a **Programming for Data Science** course.  
The notebook focuses on:

- Analysing the **time complexity** of simple algorithms.
- Using **profiling tools** to identify bottlenecks.
- Comparing **different implementations** of the same task:
  - Pure Python vs. `pandas` for CSV processing.
  - Naive vs. optimised algorithms.
  - Sequential vs. multiprocessing execution.
  - Sequential vs. multithreaded / multiprocess concurrent downloads.
- Working with **I/O-bound** and **CPU-bound** workloads in Python.

The goal is to build solid intuition about when algorithmic changes, vectorised libraries, threads or processes actually improve performance.

---

## 2. Repository structure

```text
.
├── .gitignore
├── LICENSE
├── README.md
├── Víctor_Suesta_Arribas_PEC3.ipynb   # Main notebook with all exercises
└── music_dataset.csv                  # Sample dataset used in Exercise 2
````

During execution, the notebook may also create:

```text
imgs/                                   # Folder where images downloaded from the public API are stored
```

> **Note:** Inside the notebook the CSV is referenced via a relative path (`csv_path`).
> If you change the folder structure, update `csv_path` accordingly.

---

## 3. Main notebook contents

All the work is inside `Víctor_Suesta_Arribas_PEC3.ipynb`.
The notebook is organised in five exercises:

### Exercise 1 – Duplicate detection & algorithmic complexity

* Analyses the complexity of a naive function that checks whether a list has duplicates using **nested loops**.
* Derives the worst-case complexity in **Big-O notation** (quadratic time).
* Implements an improved version using a **`set` as an auxiliary data structure** to reduce complexity to approximately **O(n)**.
* Empirically compares both implementations by:

  * Generating lists of increasing size with no duplicates.
  * Measuring average execution time for each implementation.
  * Visualising the growth using **`matplotlib`** and relating it to the theoretical complexity.

**Key concepts:** worst-case complexity, nested loops, `set` membership, empirical vs. theoretical performance.

---

### Exercise 2 – Manual CSV processing vs. pandas pipeline

* Works with `music_dataset.csv`, containing information about users of a music-streaming platform.

* Implements two functions to compute **average listening time per subscription type**:

  1. **Manual implementation**

     * Uses `open()` and manual line-by-line parsing.
     * Accumulates sums and counts in dictionaries.
     * Computes averages at the end.

  2. **pandas implementation**

     * Reads the CSV into a `DataFrame` using `pd.read_csv`.
     * Uses `groupby("subscription_type")["listening_time"].mean()` to obtain the same result.
     * Returns a Python dictionary for direct comparison with the manual version.

* Compares performance using:

  * `%timeit` to obtain average wall-clock time.
  * `line_profiler` (`%lprun`) to identify which lines dominate execution time in each function.

* Discusses how the conclusions might change if the dataset size grew by several orders of magnitude, considering both **time complexity** and **memory usage**.

**Key concepts:** file I/O, CSV parsing, `pandas` `groupby`, profiling with `%timeit` and `line_profiler`, scalability with respect to dataset size.

---

### Exercise 3 – Optimising the sum of factorials

* Starts from a deliberately inefficient implementation of:

  [
  \sum_{k=1}^{n} k!
  ]

  where each factorial is recomputed from scratch using a nested loop.

* Uses `%time` and `line_profiler` to identify the **inner loop** as the main bottleneck.

* Implements an **optimised version** that:

  * Reuses the previous factorial value (`(k - 1)!`) to compute `k!` with a single multiplication.
  * Reduces the complexity from approximately **O(n²)** to **O(n)**.

* Verifies correctness by comparing both versions on multiple test values.

* Measures the speed-up factor for a large `n`.

**Key concepts:** profiling-guided optimisation, avoiding redundant work, changing algorithmic structure vs. micro-optimisations.

---

### Exercise 4 – Sequential vs. multiprocessing factorials

* Uses a helper function based on `math.factorial` to compute factorials of large integers.

* Defines a list of large values, e.g.:

  ```python
  nums = [150000, 200000, 250000, 300000, 350000, 400000]
  ```

* Implements two strategies to compute all factorials:

  1. **Sequential version**

     * Iterates over `nums` and calls the factorial function one by one.
     * Measures total wall-clock time with `time.perf_counter()`.

  2. **Multiprocessing version**

     * Uses `multiprocessing.Pool` to distribute the computation across several processes.
     * Uses `pool.map` to apply the factorial function to all values in parallel.
     * Also measures total wall-clock time.

* Compares both runtimes and discusses:

  * When **multiprocessing** is beneficial for **CPU-bound** tasks.
  * How process-creation overhead, data serialisation and the number of cores influence performance.
  * Scenarios where a parallel implementation can be **slower** than a sequential one.

**Key concepts:** CPU-bound vs. I/O-bound tasks, `multiprocessing.Pool`, process overhead, parallel speed-up vs. problem size.

---

### Exercise 5 – Concurrent image downloads (threads vs. processes)

* Uses a small helper function `get_image_urls(num)` that calls a **public image API** (`https://picsum.photos`) and returns a list of image URLs.
* Provides a `download_image(url, idx)` function that downloads a single image and persists it to the `imgs/` folder.

The exercise develops three variants of the download service:

1. **Baseline sequential implementation**

   * Iterates over the list of 50 URLs.
   * Downloads each image one by one.
   * Measures total time as a **baseline**.

2. **Multithreading (`ThreadPoolExecutor`)**

   * Uses a thread pool with a configurable number of workers.
   * Each thread calls `download_image` for different URLs.
   * Measures total download time for worker counts from 1 to 10.

3. **Multiprocessing (`ProcessPoolExecutor`)**

   * Uses a process pool with a configurable number of workers.
   * Also measures total download time for worker counts from 1 to 10.

* Collects all timings and visualises them in a single plot:

  * X-axis: number of threads/processes.
  * Y-axis: total download time.
  * Includes a horizontal line representing the **sequential baseline**.

* Interprets the results in the context of **I/O-bound tasks**:

  * Multithreading significantly reduces total time by overlapping network waits.
  * Increasing threads beyond a certain point brings diminishing returns.
  * Multiprocessing also improves over the sequential baseline but usually performs worse than threads due to higher overhead when most of the time is spent waiting on the network.

**Key concepts:** I/O-bound workloads, `concurrent.futures`, `ThreadPoolExecutor`, `ProcessPoolExecutor`, scalability vs. number of workers.

---

## 4. How to run the notebook

You can run the notebook either in **Google Colab** or in a local Python environment.

### Option A – Google Colab (recommended for quick inspection)

1. Upload the repository contents (or clone from GitHub using the Colab terminal).
2. Open `Víctor_Suesta_Arribas_PEC3.ipynb` in Colab.
3. Adjust the `base_dir` and `csv_path` variables at the top of the notebook if your folder structure is different.
4. Run all cells (`Runtime → Run all`).

> The notebook contains a small setup block that mounts Google Drive and installs `line_profiler`.
> This is only needed when running in Colab.

### Option B – Local environment

1. Clone the repository:

   ```bash
   git clone https://github.com/Suesta/python-performance-and-concurrency.git
   cd python-performance-and-concurrency
   ```

2. (Optional but recommended) create a virtual environment.

3. Install the required packages:

   ```bash
   pip install pandas matplotlib requests line_profiler
   ```

4. Ensure that `music_dataset.csv` is in the expected path used by the notebook
   (either at the repository root or under `data/`, depending on how you configure `csv_path`).

5. Start Jupyter:

   ```bash
   jupyter notebook
   ```

   and open `Víctor_Suesta_Arribas_PEC3.ipynb`.

---

## 5. Notes on reproducibility

* **Execution times** reported in the notebook (for `%timeit`, `%time`, downloads, multiprocessing, etc.) are *indicative*.
  They naturally depend on:

  * Hardware (CPU, number of cores, RAM).
  * System load at the time of execution.
  * Network latency and bandwidth (especially in Exercise 5).
* The **qualitative conclusions** are the key point:

  * Quadratic vs. linear behaviour.
  * Relative performance of manual vs. `pandas` processing.
  * Impact of multiprocessing on CPU-bound tasks.
  * Effect of threads vs. processes for I/O-bound workloads.

---

## 6. Technologies & skills showcased

* **Python**: clean, well-documented functions with type hints and PEP8-oriented style.
* **Algorithmic thinking**: reasoning about complexity and worst-case behaviour.
* **Performance analysis**: `%timeit`, `%time`, `line_profiler`, `time.perf_counter`.
* **Data processing**: `pandas` for efficient CSV loading and aggregation.
* **Parallel and concurrent programming**:

  * `multiprocessing` module.
  * `concurrent.futures.ThreadPoolExecutor` and `ProcessPoolExecutor`.
* **Visualisation**: `matplotlib` for comparing runtimes.
* **HTTP & APIs**: downloading data from a public REST API with `requests`.

---

## 7. License

This project is released under the **MIT License**.
See the [`LICENSE`](LICENSE) file for details.

---

## 8. Author

**Víctor Suesta Arribas**

