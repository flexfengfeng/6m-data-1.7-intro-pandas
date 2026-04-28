# 🎓 Instructor Guide — Lesson 1.7: Introduction to Pandas

> **Branch:** `feature/instructor-guide`
> **Audience:** Instructors and teaching assistants
> **Companion to:** `lesson.md`, `pre-class.md`, `assignment.md`

---

## 1. Lesson Overview & Instructor Objectives

| | |
|---|---|
| **Duration** | 3 hours |
| **Format** | Flipped Classroom + Guided Coding in Jupyter/Colab |
| **Notebook** | `notebooks/pandas_lesson.ipynb` |
| **Learner entry point** | Completed 1.6 (NumPy); understands arrays; no Pandas experience |

By the end of this lesson learners should be able to:

1. Create Pandas Series and DataFrames from Python data structures.
2. Select rows and columns using `.loc` (label-based) and `.iloc` (position-based).
3. Filter DataFrames using Boolean conditions.
4. Apply functions to DataFrame columns using `.apply()` and `.map()`.
5. Sort and rank data using `.sort_values()` and `.rank()`.

**Instructor's primary job:** Pandas is the tool learners will use in *every* data project for the rest of the course and career. The investment here pays dividends in every subsequent lesson. Focus on building correct mental models for `.loc` vs. `.iloc` and `.apply()` — these are the concepts learners most frequently get wrong.

---

## 2. Concept Analogies

### Series vs. DataFrame — "Column vs. Spreadsheet"

> "A Pandas Series is a single spreadsheet column — it has an index (row labels) and values. A DataFrame is the full spreadsheet — multiple columns stitched together, sharing the same index."

**The index revelation:** Unlike NumPy arrays (0, 1, 2, 3...), Pandas indices can be *anything* — strings, dates, custom labels. Ask: "If you're tracking stock prices, what would you use as the index?" → The date. This is why Pandas is powerful for time-series data: `prices['2023-01-15']` reads as naturally as looking up a date in a calendar.

**Why does extracting one column return a Series?**
> "When you pull one column out of a DataFrame, you get a Series — just that column with its index. It's like pulling one drawer out of a filing cabinet. The drawer (Series) has all the files (values) and the labels (index), but it's no longer the whole cabinet (DataFrame)."

---

### `.loc` vs. `.iloc` — "Street Address vs. GPS Coordinates"

> "`.loc` works by *label* — give it a street address (row name or column name) and it finds the data. `.iloc` works by *position* — give it GPS coordinates (row number, column number) starting from 0, and it finds the data."

| Method | How it works | Example |
|--------|-------------|---------|
| `.loc['Alice', 'Score']` | Label-based: find row labelled 'Alice', column 'Score' | Like searching by name in a dictionary |
| `.iloc[2, 1]` | Position-based: 3rd row, 2nd column | Like accessing a grid cell by row/col number |

**The key pitfall:** When using `.iloc` for slicing, the end index is *exclusive* — `.iloc[0:3]` returns rows 0, 1, 2 (not 3). This matches Python list slicing. When using `.loc` for slicing, the end label is *inclusive*. This inconsistency trips up almost every learner — point it out explicitly.

**Which should they use?**
- Use `.loc` when you know the label (most common in data analysis)
- Use `.iloc` when you know the position (common in iteration or when working with unlabelled data)

---

### Boolean Filtering — "The Smart Filter"

> "Boolean filtering is like a custom filter in Excel, but described in code. You write a condition, and Pandas creates a list of True/False for every row. Then it keeps only the True rows."

**The chain of steps:** Make this explicit:
1. `df['Price'] > 100` → creates a Boolean Series (True/False per row)
2. `df[boolean_series]` → keeps only rows where value is True

This two-step mental model prevents learners from confusing the condition with the filter.

**Multiple conditions:** The `&` (AND) and `|` (OR) operators must have parentheses around each condition: `(df['Price'] > 100) & (df['Category'] == 'Electronics')`. Without parentheses, Python's operator precedence causes cryptic errors. Show this happening live.

---

### `.apply()` — "The Assembly Line"

> "`.apply()` puts your DataFrame on an assembly line. You define what happens at each station (your function), and apply sends every row or column through it. No loop needed — Pandas handles the iteration internally."

**vs. loops:** Show a loop-based solution and a `.apply()`-based solution side-by-side. Ask: "Which is shorter? Which is easier to read? Which is faster?" → `.apply()` wins on all three. Loops in Pandas are almost always the wrong approach.

---

## 3. Real-World Use Cases

### DataFrames in Production Data Science

Every dataset loaded from CSV, database query, Excel file, or API response lands in a DataFrame in a production data science workflow. The DataFrame is the *standard unit of data* in Python data science — understanding it thoroughly is non-negotiable.

### Boolean Filtering — Customer Segmentation

Marketing teams use Boolean filtering constantly:
```python
high_value_churned = customers[
    (customers['lifetime_value'] > 5000) & 
    (customers['days_since_last_purchase'] > 90)
]
```
This identifies high-value customers at churn risk — a standard retention marketing query.

### `.apply()` — Feature Engineering

In machine learning, feature engineering (creating new columns from existing data) almost always uses `.apply()`:
```python
df['age_group'] = df['age'].apply(lambda x: 'Young' if x < 35 else 'Mature')
df['email_domain'] = df['email'].apply(lambda x: x.split('@')[1])
df['risk_score'] = df.apply(lambda row: calculate_risk(row['income'], row['debt']), axis=1)
```

### Sorting and Ranking — Leaderboards and Reports

Finance teams use `.rank()` to create relative rankings within groups — "which salesperson ranks top by revenue in each region this quarter?" The game leaderboard activity in Part 3 is a direct analogy.

---

## 4. Activity Facilitation Notes

### Part 1: The Inventory System (60 min)

**Start with creation, not loading.** Learners often spend their first Pandas sessions confused about where data comes from. Building a DataFrame from scratch (from a dictionary) gives them full control and visibility.

**The vectorisation discussion is important:**
Ask: "In Python, if you want to multiply all prices by 1.2, how would you do it without Pandas?" → A loop: `for price in prices: price * 1.2`. Then show: `df['Price'] * 1.2`. Ask: "What just happened?" → The operation applied to the entire column at once, no loop. This is vectorisation. Connect it back to NumPy (Lesson 1.6) — Pandas column operations *are* NumPy operations under the hood.

**Adding a new column:** Two ways:
- Scalar: `df['Discount'] = 0.1` — all rows get the same value
- Derived: `df['Discounted_Price'] = df['Price'] * 0.9` — each row gets its own value

Show both. Ask: "When would you use the scalar version?" → Default values, flags, constant offsets.

---

### Part 2: The Data Detective (60 min)

**`.loc` vs `.iloc` demonstration:**
Create a small DataFrame with string index labels (not default 0, 1, 2) so the distinction between label and position is obvious:
```python
df = pd.DataFrame({'Score': [85, 72, 90]}, index=['Alice', 'Bob', 'Carol'])
```
Then ask:
- "Select Alice's score with `.loc`" → `df.loc['Alice', 'Score']`
- "Select the first row with `.iloc`" → `df.iloc[0]`
- "Are these the same?" → Yes, because Alice is in position 0. "Now if we sort the index, are they still the same?" → No — `.loc` still finds Alice, `.iloc[0]` now finds whoever is first alphabetically.

This is the clearest demonstration of why the distinction matters.

**Boolean filter mistake to surface proactively:**
Ask learners to filter for scores above 80. Watch for those who write:
```python
df[df.Score > 80]  # Works but fragile — column names with spaces break this
```
vs.
```python
df[df['Score'] > 80]  # Preferred — always works
```
Address both forms, but recommend the bracket notation as the professional standard.

---

### Part 3: Leaderboard Logic (60 min)

**Custom functions with `.apply()`:**
Walk through the function-definition-first, apply-second pattern:
1. Define the categorisation function and test it on a single value first.
2. Then `.apply()` it to the column.

This "test before applying" habit prevents apply mistakes where the function works on one value but fails on the column.

**Sorting by index vs. values:**
Ask: "What's the default sort order of a DataFrame?" → By index (insertion order, usually). "When would you want to sort by values instead?" → Ranking, leaderboards, finding top/bottom performers.

**The `.rank()` tie-handling discussion:**
Show the `method` parameter options: `'average'`, `'min'`, `'max'`, `'dense'`, `'first'`. Connect back to SQL RANK vs DENSE_RANK (Lesson 1.5). Ask: "Which ranking method would be fairest in a student grade report?" → `'dense'` or `'min'` typically (no gaps, ties get same rank). "Which would you use for a sports leaderboard where the prize money depends on exact position?" → `'first'` (deterministic ordering) or `'average'`.

---

## 5. Timing & Pacing Notes

| Part | Planned | Common Overrun | Mitigation |
|------|---------|---------------|-----------|
| Part 1: Structures | 60 min | DataFrame creation from various sources takes time to demonstrate | Stick to dict-based creation; show CSV loading briefly as preview for 1.8 |
| Part 2: Selection | 60 min | `.loc` vs `.iloc` confusion generates many questions | Use the string-indexed DataFrame example early; return to it whenever confusion arises |
| Part 3: Analysis | 60 min | `.apply()` with multi-column (axis=1) confuses learners | Keep axis=1 examples as optional stretch; focus on single-column apply |

---

## 6. Common Learner Questions

**Q: "Why do I get a warning about 'SettingWithCopyWarning'?"**
A: Pandas warns you when you're modifying what might be a *view* (like a DataFrame slice) rather than the original. To safely modify a subset, use `.copy()` on the slice first, or use `.loc` to modify the original directly. This is the Pandas equivalent of the NumPy view vs. copy issue.

**Q: "What's the difference between `.apply()` and `.map()`?"**
A: `.map()` works on Series only and applies element-wise. `.apply()` works on both Series and DataFrames, and can operate element-wise or row/column-wise. For simple element transformations on a single column, both work. Use `.apply()` when you need access to multiple columns (axis=1).

**Q: "How do I apply a function to multiple columns at once?"**
A: Use `df.apply(func, axis=1)` where `func` receives each row as a Series. Inside the function, access columns by name: `row['column_name']`. This is the most flexible but also slowest approach — for production code, vectorized operations are preferred.

**Q: "When should I NOT use `.apply()`?"**
A: When a vectorized operation exists that does the same thing — vectorized operations (arithmetic, string methods with `.str.`, datetime with `.dt.`) are always faster than `.apply()`. Use `.apply()` for custom logic that can't be expressed as a vectorized operation.
