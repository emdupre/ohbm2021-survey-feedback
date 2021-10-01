---
jupytext:
  formats: md:myst,ipynb
  text_representation:
    extension: .md
    format_name: myst
    format_version: '0.9'
    jupytext_version: 1.5.2
kernelspec:
  display_name: Python 3
  name: python3
---

# OHBM2021 Survey Feedback

This notebook contains code for rendering survey results from the OHBM 2021 Annual Meeting Evaluation.
It is intended to accompany a recent [OHBM Blog](https://www.ohbmbrainmappingblog.com/) post exploring member feedback for the 2021 virtual event.

````{margin}
```{note}
All content is written in [MyST](https://jupyterbook.org/content/myst.html) and rendered using [Jupyer Book](https://jupyterbook.org/intro.html).
To ease navigation, several code cells are hidden by default.
You can expand these code cells by clicking on the 'Click to show' buttons.

You can also load the environment to further explore these data by clicking on the `Binder` button:

[![Binder](https://mybinder.org/badge_logo.svg)](https://mybinder.org/v2/gh/emdupre/ohbm2021-survey-feedback/HEAD?filepath=content%2Findex.md)

This will launch a new [MyBinder](https://mybinder.org) session allowing you to re-analyze these data directly in the browser.
```
````

```{code-cell} python3
:tags: ["hide-cell"]
import numpy as np
import pandas as pd
from textwrap import wrap

import matplotlib.pyplot as plt
plt.rcParams.update({'font.size': 16})

import seaborn as sns
sns.set(context='talk', style='white')

import warnings
warnings.filterwarnings('ignore')
```

We'll use a limited subset of the full survey data.
Data were collected on a Likert scale from 1-5, where 1 indicates 'very poor' experience with a given aspect of the annual meeting and 5 indicates an 'excellent' experience.

```{code-cell} python3
df = pd.read_csv('./ohbm2021-annual-meeting-eval.csv')
```

We'll define a new function `plot_stacked_bar` to generate a stacked barplot showing the distribution of responses for each question.
We can then apply this function to the loaded pandas dataframe and plot the results.

```{code-cell} python3
:tags: ["hide-input"]
def plot_stacked_bar(df, figwidth=25, textwrap=30):
    """
    A wrapper function to create a stacked bar plot.
    Seaborn does not implement this directly, so
    we'll use seaborn styling in matplotlib.

    Parameters
    ----------
    figwidth: float
        The desired width of the figure. Also controls
        spacing between bars.
    textwrap: int
        The number of characters (including spaces) allowed
        on a line before wrapping to a newline.
    """
    reshape = pd.melt(df, var_name='option', value_name='rating')
    stack = reshape.rename_axis('count').reset_index().groupby(['option', 'rating']).count().reset_index()

    fig, ax = plt.subplots(1, 1, figsize=(15, figwidth))
    bottom = np.zeros(len(stack['option'].unique()))
    clrs = sns.color_palette('Set1', n_colors=5)  # to do: check colorblind friendly-ness
    labels = ['Very poor', 'Poor', 'Indifferent', 'Good', 'Very good']

    for rating in range(1, 6):
        stackd = stack.query(f'rating == {rating}')
        ax.barh(y=stackd['option'], width=stackd['count'], left=bottom,
                tick_label=['\n'.join(wrap(s, textwrap)) for s in stackd['option']],
                color=clrs[rating - 1], label=labels[rating - 1])
        bottom += np.asarray(stackd['count'])

    sns.despine()
    ax.set_xlabel('Count', labelpad=20)
    ax.legend(title='Rating', bbox_to_anchor=(1, 1))

    return ax
```

```{code-cell} python3
:tags: ["full-width"]
ax = plot_stacked_bar(df)
fig = ax.figure
fig.show()
```

We can see that 'Experience accessing pre-recorded content in the Screening Room` had the highest proportion of responses indicating an excellent experience,
while 'Virtual meeting website navigation' had the highest number of respondents with a 'poor' experience.