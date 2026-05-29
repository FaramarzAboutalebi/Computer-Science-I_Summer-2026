# Computer Science I, Summer 2026 Second Session

# Q1

**Q:** An `int` pointer `arr` points to a dynamically allocated `int` array of size `n` that is completely full. Write a line of code that doubles the size of the array without losing any data.

```c
arr = realloc(arr, n * 2 * sizeof(int));
```

**Explanation:** `realloc` resizes a previously allocated block. It keeps the existing `n` ints intact and extends the block to hold `2n` ints. If it has to move the block to find enough contiguous space, it copies the old data over for you, so nothing is lost.

> Technically it is safer to first check whether `realloc` returns `NULL` before assigning it back to `arr` (otherwise a failed realloc leaks the original block). But since the question asks for a single line of code, the line above is the expected answer. The safer version would be:
>
> ```c
> int *tmp = realloc(arr, n * 2 * sizeof(int));
> if (tmp != NULL) arr = tmp;
> ```

---

# Q2

**Q:** Given the structure below, complete a function that takes an array of student names, an array of corresponding scores, and the count `n`. Return a dynamically allocated array of `n` pointers to `Student`. Each `Student` is separately allocated, and each name is dynamically allocated (exact space needed) and copied from the `names` array.

```c
typedef struct Student_s {
    char *name;
    int   score;
} Student;
```

```c
Student **createRoster(char names[][101], int scores[], int n) {
    // allocate Student* array
    Student **roster = malloc(n * sizeof(Student*));

    // for each student
    for (int i = 0; i < n; i++) {
        // allocate memory for the Student
        roster[i] = malloc(sizeof(Student));

        // allocate space to store name properly
        int len = strlen(names[i]);
        roster[i]->name = malloc((len + 1) * sizeof(char));
        strcpy(roster[i]->name, names[i]);   // copying name

        // save the score
        roster[i]->score = scores[i];        // copying score
    }
    return roster;
}
```

**Explanation:** There are three separate levels of allocation here, which is why the diagram below has three "layers."

1. **The pointer array** — `malloc(n * sizeof(Student*))` allocates one contiguous block holding `n` pointers. `roster` points to it.
2. **Each `Student` struct** — inside the loop, `roster[i] = malloc(sizeof(Student))` allocates one struct per student. Each `roster[i]` points to its own struct.
3. **Each name string** — `strlen(names[i])` gives the length *without* the null terminator, so we allocate `len + 1` bytes to leave room for `'\0'`. `strcpy` then copies the characters plus the terminator. This is what "exact space needed" means — no wasted bytes.

The `score` is a plain `int` living inside the struct, so it is just assigned directly (no allocation needed).

### Memory diagram (for `n = 3`)

```
   roster
  (Student**)
      │
      ▼
 ┌───────────┬───────────┬───────────┐   <- malloc(n * sizeof(Student*))
 │ roster[0] │ roster[1] │ roster[2] │      one block of n pointers
 └─────┬─────┴─────┬─────┴─────┬─────┘
       │           │           │
       ▼           ▼           ▼
  ┌─────────┐ ┌─────────┐ ┌─────────┐       <- each malloc(sizeof(Student))
  │ name  ──┼ │ name  ──┼ │ name  ──┼            one Student struct each
  │ score=88│ │ score=91│ │ score=75│
  └────┬────┘ └────┬────┘ └────┬────┘
       │           │           │
       ▼           ▼           ▼
   ┌─────────┐ ┌──────┐  ┌────────────┐    <- each malloc(len + 1)
   │A│l│i│\0│ │B│o│\0│  │C│a│r│o│l│\0│        exact-size name strings
   └─────────┘ └──────┘  └────────────┘
```

> **Note on freeing:** because there are three allocation layers, freeing must happen in reverse order — first each `roster[i]->name`, then each `roster[i]`, then `roster` itself. Freeing only `roster` would leak all the structs and names.

---

# Q3.a

## First approach

| i | j |
|---|---|
| 0 | n |
| 1 | n-1 |
| 2 | n-2 |
| ⋮ | ⋮ |
| n-1 | n-(n-1) = 1 |

$$n + (n-1) + (n-2) + \cdots + 1 = \sum_{i=1}^{n} i = \frac{n(n+1)}{2}$$

$$= \frac{n^2+n}{2} \longrightarrow O(n^2)$$

## Second approach

$$\sum_{i=0}^{n-1} \sum_{j=0}^{n-i-1} 1 = \sum_{i=0}^{n-1}\left(\sum_{j=1}^{n-i-1} 1 - \sum_{j=1}^{-1} 1\right)$$

$$= \sum_{i=0}^{n-1}\left(n-i-1 \;+\; 1\right) = \sum_{i=0}^{n-1}(n-i)$$

$$= \sum_{i=0}^{n-1} n \;-\; \sum_{i=0}^{n-1} i = \sum_{i=1}^{n-1} n - \sum_{i=1}^{-1} n - \left(\sum_{i=1}^{n-1} i - \sum_{i=1}^{-1} i\right)$$

$$= (n-1)n - \big(n(-1)\big) - \frac{(n-1)n}{2} - 0 = n^2 - \cancel{n} + \cancel{n} = \frac{n^2-n}{2}$$

$$= \frac{2n^2 - n^2 + n}{2} = \frac{n^2+n}{2} \longrightarrow O(n^2) \qquad \boxed{Q3.a}$$

---

# Q3.b

## First approach

$$\boxed{1,\ 2,\ 3,\ \ldots,\ n} \longrightarrow O(n)$$

$$\boxed{\phantom{xxxxxxxxxx}} \longrightarrow\ ?\ \ O(\log n)$$

$$\boxed{0,\ 1,\ 2,\ \ldots,\ n-1} \longrightarrow O(n)$$

| iteration | j |
|-----------|---|
| 1 | $1 = 2^0$ |
| 2 | $4 = 2^2$ |
| 3 | $8 = 2^3$ |
| ⋮ | |
| t | $2^t$ |

$$\text{when loop stops?}\quad j > n \rightarrow 2^t > n \rightarrow \log_2 2^t > \log_2 n$$

$$\rightarrow t > \log_2 n \rightarrow O(\log n)$$

$$\text{so } O(n * n * \log n) = O(n^2 \log n)$$

## Second approach

$$\sum_{i=1}^{n} \sum_{j=0}^{\lg n} \sum_{k=0}^{n-1}$$

$$j = 2^0,\ 2,\ 4,\ \ldots,\ 2^m$$
$$2^m <= n \rightarrow m <= \log n$$

$$\sum_{i=1}^{n} \sum_{j=0}^{\log n} \sum_{k=0}^{n-1} 1 = \sum_{i=1}^{n} \sum_{j=0}^{\log n}\left(\sum_{k=1}^{n-1} 1 - \sum_{k=1}^{-1} 1\right)$$

$$= \sum_{i=1}^{n} \sum_{j=0}^{\log n} (n-1+1) = \sum_{i=1}^{n} \sum_{j=0}^{\log n} n$$

$$= \sum_{i=1}^{n}\left(\sum_{j=1}^{\log n} n - \sum_{j=1}^{-1} n\right) = \sum_{i=1}^{n}(n\log n + n)$$

$$= (n\log n + n)\sum_{i=1}^{n} 1 = n(n\log n + n) = n^2\log n + n^2$$

$$\longrightarrow O(n^2 \log n)$$

---

# Q3.c

## First approach

| iteration | m |
|-----------|---|
| 1 | $m/2$ |
| 2 | $m/2^2$ |
| 3 | $m/2^3$ |
| ⋮ | |
| t | $m/2^t$ |

$$\text{when loop stops?}\quad m/2^t < 1 \rightarrow m < 2^t \rightarrow \log m < t \rightarrow O(\log m)$$

## Second approach

$$\sum_{k=0}^{\log m} 1 = \sum_{k=1}^{\log m} 1 - \sum_{k=1}^{-1} 1 = \log m + 1 \longrightarrow O(\log m)$$

$$\frac{m}{2^0}$$
$$\frac{m}{2^1}$$
$$\frac{m}{2^2}$$
$$\frac{m}{2^3}$$
$$\vdots$$
$$\boxed{\dfrac{m}{2^k}}$$

$$\frac{m}{2^k} >= 1 \rightarrow m >= 2^k \rightarrow k <= \log m$$

---

# Q4

**Q:** Put the following Big-O running times in order from best to worst:

$$O(n^2),\ O(\log n),\ O(1),\ O(2^n),\ O(n \log n),\ O(n^3),\ O(n)$$

**Answer (best → worst):**

$$O(1) < O(\log n) < O(n) < O(n \log n) < O(n^2) < O(n^3) < O(2^n)$$

**Explanation:** As $n$ grows, slower-growing functions are "better." Constant time never grows; logarithmic grows extremely slowly; linear grows in step with $n$; $n\log n$ is slightly worse than linear; then the polynomials $n^2$ and $n^3$ grow faster the higher the exponent; and the exponential $2^n$ eventually dwarfs every polynomial.

| Rank | Complexity | Name |
|------|------------|------|
| 1 (best) | $O(1)$ | constant |
| 2 | $O(\log n)$ | logarithmic |
| 3 | $O(n)$ | linear |
| 4 | $O(n \log n)$ | linearithmic |
| 5 | $O(n^2)$ | quadratic |
| 6 | $O(n^3)$ | cubic |
| 7 (worst) | $O(2^n)$ | exponential |

---

# Q5

$$\sum_{i=2}^{2n}(i + 2i^2) = \sum_{i=2}^{2n} i + 2\sum_{i=2}^{2n} i^2$$

$$= \sum_{i=1}^{2n} i - \sum_{i=1}^{1} i + \left(2\sum_{i=1}^{2n} i^2 - 2\sum_{i=1}^{1} i^2\right)$$

$$= \frac{2n(2n+1)}{2} - \frac{1(1+1)}{2} + 2\,\frac{2n(2n+1)(4n+1)}{6} - 2*\frac{1(2)(3)}{6}$$

$$= n(2n+1) - 1 + \frac{2n(2n+1)(4n+1)}{3} - 2$$
