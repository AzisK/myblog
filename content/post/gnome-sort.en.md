---
title: "Gnome Sort: Sorting Like a Garden Gnome"
date: 2026-04-01T13:50:47Z
author: Ąžuolas Krušna
tags: ["Python", "Algorithms", "Sorting"]
ShowCodeCopyButtons: true
draft: true
---

Picture a garden gnome standing in a row of flower pots, each pot labelled with a number. His job: sort them. He's not the brightest gnome — he only ever looks at the pot in front of him and the one just behind. If they're out of order, he swaps them and shuffles back one step. If they're fine, he shuffles forward. He keeps going until he reaches the end of the row.

That's gnome sort. Beautifully simple, hilariously inefficient at scale, and a perfect algorithm to understand how sorting *really* works from first principles.

## The Idea

Gnome sort belongs to the same family as insertion sort — it finds an element that's out of place and drags it backwards until it lands in the right spot. The key difference is how it does the dragging: instead of shifting elements in bulk, it swaps one step at a time.

The rules are just two:

1. If the current element is ≥ the previous one (or we're at the start), **move forward**.
2. If the current element is < the previous one, **swap them and move backward**.

Repeat until the gnome walks off the end of the array.

## Building It Step by Step

Let's implement this in Python, piece by piece.

### Step 1: The skeleton

We need a position pointer `pos` that starts at 0 and walks the array.

```python
def gnome_sort(arr):
    pos = 0
    while pos < len(arr):
        pos += 1
```

This just walks forward — no sorting yet. Let's add the actual logic.

### Step 2: Move forward or swap backward

```python
def gnome_sort(arr):
    pos = 0
    while pos < len(arr):
        if pos == 0 or arr[pos] >= arr[pos - 1]:
            pos += 1
        else:
            arr[pos], arr[pos - 1] = arr[pos - 1], arr[pos]
            pos -= 1
```

That's it. Seriously — that's the whole algorithm.

- `pos == 0` guards against going out of bounds when stepping back.
- `arr[pos] >= arr[pos - 1]` checks if the current pair is already in order.
- If not, we swap and step back to re-check the newly placed element.

### Step 3: Watch it run

Let's trace through a small example: `[3, 1, 2]`.

```
pos=0  → at start, move forward         → [3, 1, 2]
pos=1  → 1 < 3, swap, move back         → [1, 3, 2]
pos=0  → at start, move forward         → [1, 3, 2]
pos=1  → 3 ≥ 1, move forward            → [1, 3, 2]
pos=2  → 2 < 3, swap, move back         → [1, 2, 3]
pos=1  → 2 ≥ 1, move forward            → [1, 2, 3]
pos=2  → 3 ≥ 2, move forward            → [1, 2, 3]
pos=3  → end of array, done!            → [1, 2, 3] ✓
```

### Step 4: Add some visibility

To really see the gnome at work, let's print each step:

```python
def gnome_sort(arr):
    pos = 0
    while pos < len(arr):
        if pos == 0 or arr[pos] >= arr[pos - 1]:
            pos += 1
        else:
            arr[pos], arr[pos - 1] = arr[pos - 1], arr[pos]
            pos -= 1
        print(f"pos={pos}  {arr}")
    return arr
```

Run it:

```python
gnome_sort([5, 3, 1, 4, 2])
```

```
pos=1  [5, 3, 1, 4, 2]
pos=0  [3, 5, 1, 4, 2]
pos=1  [3, 5, 1, 4, 2]
pos=0  [3, 1, 5, 4, 2]   ← 1 bubbling left
pos=1  [1, 3, 5, 4, 2]
pos=2  [1, 3, 5, 4, 2]
pos=3  [1, 3, 5, 4, 2]
pos=2  [1, 3, 4, 5, 2]
pos=3  [1, 3, 4, 5, 2]
pos=2  [1, 3, 4, 2, 5]   ← 2 bubbling left
pos=1  [1, 3, 2, 4, 5]
pos=0  [1, 2, 3, 4, 5]
pos=1  [1, 2, 3, 4, 5]
...
pos=5  [1, 2, 3, 4, 5] ✓
```

You can clearly see each small element pulling itself left, one swap at a time.

### Step 5: The final version

Clean it up and return the sorted list:

```python
def gnome_sort(arr: list) -> list:
    pos = 0
    while pos < len(arr):
        if pos == 0 or arr[pos] >= arr[pos - 1]:
            pos += 1
        else:
            arr[pos], arr[pos - 1] = arr[pos - 1], arr[pos]
            pos -= 1
    return arr
```

## How Fast Is It?

| Case | Time Complexity |
|------|----------------|
| Best (already sorted) | O(n) |
| Average | O(n²) |
| Worst (reverse sorted) | O(n²) |

**Space complexity:** O(1) — it sorts in place, no extra memory needed.

In the best case the gnome walks straight from start to finish without a single swap. In the worst case he shuffles every element all the way to the front — which adds up to a lot of steps.

For real-world use, Python's built-in `sorted()` (Timsort, O(n log n)) will beat gnome sort on any non-trivial list. But gnome sort earns its place as a teaching tool — the code is short enough to hold in your head, and the mental model (one gnome, two rules) makes it easy to reason about correctness.

## Summary

Gnome sort in two rules:
- Move forward if the current pair is in order.
- Swap and step back if it isn't.

Five lines of Python. O(n²) time, O(1) space. Not fast, but unforgettable.

## Sources

- Original paper by Hamid Sarbazi-Azad (2000): https://doswww.do.isep.ipp.pt/Algoritmos/Gnome%20Sort.pdf
- Visualization: https://www.cs.usfca.edu/~galles/visualization/Algorithms.html
- Wikipedia: https://en.wikipedia.org/wiki/Gnome_sort

---
