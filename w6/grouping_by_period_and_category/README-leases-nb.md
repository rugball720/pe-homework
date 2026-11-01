---
jupytext:
  custom_cell_magics: kql
  text_representation:
    extension: .md
    format_name: myst
    format_version: 0.13
    jupytext_version: 1.19.5
kernelspec:
  display_name: Python 3 (ipykernel)
  language: python
  name: python3
---

(label-tp-leases)=
# grouping by period and category

```{admonition} download the zip
:class: warning

to work on this assignment locally on your laptop, {download}`start with downloading the zip<./ARTEFACTS-leases.zip>`
```

in this TP we work on

- data that represents *periods* and not just one timestamp
- checking for overlaps
- grouping by period (week, month, year..)
- then later on, grouping by period *and* category
- and some simple visualization tools

here's an example of the outputs we will obtain

(label-leases-output)=
````{grid} 3 3 3 3
```{image} media/result-color-w.png
```
```{image} media/result-color-m.png
```
```{image} media/result-color-y.png
```
````

+++

## imports

```{code-cell} ipython3
import pandas as pd
import matplotlib.pyplot as plt
```

1. make sure to use matplotlib in interactive mode - aka `ipympl`

```{code-cell} ipython3
:tags: [level_basic]

%matplotlib ipympl
```

2. optional: setup itables, so that we can have scrollable tables

```{code-cell} ipython3
:tags: [level_basic]

try:
    from itables import init_notebook_mode
    init_notebook_mode(all_interactive=True)
except ImportError:
    pass
```

## the data

we have a table of events, each with a begin (`beg`) and `end` time; in addition each is attached to a `country`

```{code-cell} ipython3
leases = pd.read_csv("data/leases.csv")
leases.head(10)
```

### adapt the type of each columns

surely the columns dtypes need some care

```{code-cell} ipython3
:tags: [level_basic]

leases['beg'] = pd.to_datetime(leases['beg'])
leases['end'] = pd.to_datetime(leases['end'])
```

```{code-cell} ipython3
:tags: [level_intermediate]

# check it

leases.dtypes
```

### raincheck

check that the data is well-formed, i.e. **the `end`** timestamp **happens after `beg`**

```{code-cell} ipython3
:tags: [level_basic]

(leases['end'] > leases['beg']).all()
```

### are there any overlapping events ?

+++

it turns out there are **no event overlap**, but write a code that checks that this is true

```{admonition} note
:class: tip

nothing in the rest depends on this question, so if you find this too hard, you can skip to the next question
```

```{code-cell} ipython3
:tags: [level_basic]

leases = leases.sort_values('beg')
(leases['beg'].shift(-1) >= leases['end']).dropna().all()
```

### timespan

What is the timespan covered by the dataset (**earliest** and **latest** events, and **duration** in-between) ?

```{code-cell} ipython3
:tags: [level_basic]

earliest = leases['beg'].min()
latest = leases['end'].max()
duration = latest - earliest
earliest, latest, duration
```

### aggregated duration

so, given that there is no overlap, we can assume this corresponds to "reservations" attached to a unique resource (hence the term  *lease*)  
write a code that computes the **overall reservation time**, as well as the **average usage ratio** over the overall timespan

```{code-cell} ipython3
:tags: [level_basic]

leases['duration'] = leases['end'] - leases['beg']
total_time = leases['duration'].sum()
ratio = total_time / duration
total_time, ratio
```

## visualization - grouping by period

### usage by period

grouping by periods: by week, by month or by year, display the **total usage in that period**  
(when ambiguous, use the `beg` column to determine if a lease is in a period or the other)

```{admonition} *hint*
:class: dropdown tip

There are at least 2 options to do this grouping, based on `resample()` and `to_period()`  
advanced users may wish to write them both and to comment on their respective pros and cons

```

`````{admonition} for now, **just get the grouping right**
:class: dropdown

you should produce something like e.g.

````{grid} 3 3 3 3
```{image} media/result-bw-w.png
```
```{image} media/result-bw-m.png
```
```{image} media/result-bw-y.png
```
````
we'll make cosmetic improvements below, and [the final results look like this](#label-leases-output), but let's not get ahead of ourselves
`````

```{code-cell} ipython3
:tags: [level_basic]

leases_w = leases.set_index('beg').resample('W')['duration'].sum()
leases_m = leases.set_index('beg').resample('ME')['duration'].sum()
leases_y = leases.set_index('beg').resample('YE')['duration'].sum()

fig, axes = plt.subplots(1, 3, figsize=(15, 4))
leases_w.plot(kind='bar', ax=axes[0])
leases_m.plot(kind='bar', ax=axes[1])
leases_y.plot(kind='bar', ax=axes[2])
plt.show()
```

### improve the title and bottom ticks

add a title to your visualisations

also, and particularly relevant in the case of the per-week visu, we don't get to read **the labels on the horizontal axis**, because there are **too many of them**  
to improve this, you can use matplotlib's `set_xticks()` function; you can either figure out by yourself, or read the few tips below

````{admonition} a few tips
:class: dropdown tip

- the object that receives the `set_xticks()` method is an instance of `Axes` (one X&Y axes system),  
  which is not the figure itself (a figure may contain several Axes)  
  ask google or chatgpt to find the way you can spot the `Axes` instance in your figure
- it is not that clear in the docs, but all you need to do is to pass `set_xticks` a list of *indices* (integers)  
  i.e. if you have, say, a hundred bars, you could pass `[0, 10, 20, ..., 100]` and you will end up with one tick every 10 bars.
- there are also means to use smaller fonts, which may help see more relevant info
````

```{code-cell} ipython3
# let's say as arule of thumb
LEGEND = {
    'W': "week",
    'M': "month",
    'Y': "year",
}

SPACES = {
    'W': 12,   # in the per-week visu, show one tick every 12 - so about one every 3 months
    'M': 3,    # one every 3 months
    'Y': 1,    # on all years
}
```

```{code-cell} ipython3
:tags: [level_basic]

fig, axes = plt.subplots(1, 3, figsize=(15, 4))

leases_w.plot(kind='bar', ax=axes[0])
axes[0].set_title(LEGEND['W'])
axes[0].set_xticks(range(0, len(leases_w), SPACES['W']))

leases_m.plot(kind='bar', ax=axes[1])
axes[1].set_title(LEGEND['M'])
axes[1].set_xticks(range(0, len(leases_m), SPACES['M']))

leases_y.plot(kind='bar', ax=axes[2])
axes[2].set_title(LEGEND['Y'])
axes[2].set_xticks(range(0, len(leases_y), SPACES['Y']))

plt.show()
```

### a function to convert to hours

you are to write a function that converts a `pd.Timedelta` into a number of hours  
1. read and understand the test code for the details of what is expected
2. use it to test your own implementation

```{code-cell} ipython3
:tags: [level_basic]

# your code

def convert_timedelta_to_hours(timedelta):
    s = timedelta.total_seconds()
    hours = int(s // 3600)
    if s % 3600 > 0:
        hours += 1
    return hours
```

```{code-cell} ipython3
:tags: [level_intermediate]

# test it

# if an hour has started even by one second, it is counted
test_cases = ( 
    # input in seconds, expected result in hours
    (0, 0), 
    (1, 1),     (3599, 1),     (3600, 1), 
    (3601, 2),  (7199, 2),     (7200, 2), 
    # 2 hours + 1s -> 3 hours
    (7201, 3),  
    # 3 hours + 2 minutes -> 4 hours
    (pd.Timedelta(3, 'h') + pd.Timedelta(2, 'm'), 4),
    # 2 days -> 48 hours
    (pd.Timedelta(2, 'D'), 48),
)

def test_convert_timedelta_to_hours():
    for seconds, exp in test_cases:
        # convert into pd.Timedelta if not already one
        if not isinstance(seconds, pd.Timedelta):
            timedelta = pd.Timedelta(seconds=seconds)
        else:
            timedelta = seconds
        # compute and compare
        got = convert_timedelta_to_hours(timedelta)
        print(f"with {timedelta=} we get {got} and expected {exp} -> {got == exp}")

test_convert_timedelta_to_hours()
```

```{code-cell} ipython3
:tags: [level_intermediate]

# for debugging; this should return 48

convert_timedelta_to_hours(pd.Timedelta(2, 'D'))
```

### use it to display totals in hours

keep the same visu, but display **the Y axis in hours**  
btw, what was the unit in the graphs above ?

```{code-cell} ipython3
:tags: [level_basic]

leases['hours'] = leases['duration'].apply(convert_timedelta_to_hours)

leases_w_h = leases.set_index('beg').resample('W')['hours'].sum()
leases_w_h.index = leases_w_h.index.strftime('%m-%d')

leases_m_h = leases.set_index('beg').resample('ME')['hours'].sum()
leases_m_h.index = leases_m_h.index.strftime('%y-%m')

leases_y_h = leases.set_index('beg').resample('YE')['hours'].sum()
leases_y_h.index = leases_y_h.index.strftime('%Y')

fig, axes = plt.subplots(1, 3, figsize=(15, 4))

leases_w_h.plot(kind='bar', ax=axes[0])
axes[0].set_title("Duration in hours per week")
axes[0].set_xticks(range(0, len(leases_w_h), SPACES['W']))

leases_m_h.plot(kind='bar', ax=axes[1])
axes[1].set_title("Duration in hours per month")
axes[1].set_xticks(range(0, len(leases_m_h), SPACES['M']))

leases_y_h.plot(kind='bar', ax=axes[2])
axes[2].set_title("Duration in hours per year")
axes[2].set_xticks(range(0, len(leases_y_h), SPACES['Y']))

plt.show()
```

## grouping by period and region

the following table allows you to map each country into a region

```{code-cell} ipython3
# load it

countries = pd.read_csv("data/countries.csv")
countries.head(3)
```

### a glimpse on regions

what's the most effective way to see how many regions and how many countries per region ?

```{code-cell} ipython3
:tags: [level_basic]

countries['region'].value_counts()
```

### attach a region to each lease

your mission is to now show the same graphs, but we want to reflect the relative usage of each region, so we want to [split each bar into several colors, one per region see expected result below](#label-leases-output)

+++

most likely your first move is to tag all leases with a `region` column

```{code-cell} ipython3
:tags: [level_basic]

leases = leases.merge(countries, left_on='country', right_on='name', how='left')
```

### visu by period by region

you can now produce [the target figures, again they look like this](#label-leases-output)

```{code-cell} ipython3
:tags: [level_basic]

df_w = leases.pivot_table(index=pd.Grouper(key='beg', freq='W'), columns='region', values='hours', aggfunc='sum').fillna(0)
df_w.index = df_w.index.strftime('%m-%d')

df_m = leases.pivot_table(index=pd.Grouper(key='beg', freq='ME'), columns='region', values='hours', aggfunc='sum').fillna(0)
df_m.index = df_m.index.strftime('%y-%m')

df_y = leases.pivot_table(index=pd.Grouper(key='beg', freq='YE'), columns='region', values='hours', aggfunc='sum').fillna(0)
df_y.index = df_y.index.strftime('%Y')

fig, axes = plt.subplots(1, 3, figsize=(15, 4))

df_w.plot(kind='bar', stacked=True, ax=axes[0])
axes[0].set_title("Duration in hours per week")
axes[0].set_xticks(range(0, len(df_w), SPACES['W']))

df_m.plot(kind='bar', stacked=True, ax=axes[1])
axes[1].set_title("Duration in hours per month")
axes[1].set_xticks(range(0, len(df_m), SPACES['M']))

df_y.plot(kind='bar', stacked=True, ax=axes[2])
axes[2].set_title("Duration in hours per year")
axes[2].set_xticks(range(0, len(df_y), SPACES['Y']))

plt.show()
```

***
