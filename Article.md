# Creating Custom Graphs with Google Trends and pandas

Perhaps you're interested in using Google Trends but find the website rather limiting; with the help of this article you will learn how to create custom graphs using Google Trends data.

We will use [Pytrends](https://pypi.org/project/pytrends/) (an unofficial Google Trends API), as well as the usual suspects in visualizations: [Matplotlib](https://matplotlib.org) and specifically [pandas](https://pandas.pydata.org/docs/getting_started/index.html#getting-started), as it has integrated functionality for Matplotlib. We'll work in [Jupyter Notebook](https://jupyter.org), with [ReviewNB](https://www.reviewnb.com/) as our source control solution.

This article assumes you have _some_ prior experience using Jupyter notebooks, GitHub and Matplotlib/pandas.

Let's get started by creating a new notebook.

## Setting up and connecting to Google

Install the Pytrends module: run `pip install pytrends` on your anaconda prompt, or `!pip install pytrends` on the notebook itself. Give it a few seconds and you will see the output once it is done.

Now we can import all the modules we will need with the following code:

```
import matplotlib.pyplot as plt
import pandas as pd
import pytrends
from pytrends.request import TrendReq
```

Connect to Google Trends using the `TrendReq` class imported from `pytrends.request` using this code:

```
pytrends = TrendReq(hl='en-US')
```

This initializes our Pytrends object and establishes a connection to Google Trends. The `hl`(host language) is set to American English.

## Creating a bar chart: Interest Over Time

Now we can get Google Trends data for our first graph. For this example, we'll plot the data for the search terms "Java", "Python" and "Javascript" over a given timeframe into a bar chart to illustrate Google users' interest in the topics over time.

Type in the following:

```
kw_list = ["Java", "Python", "Javascript"]
pytrends.build_payload(kw_list, geo='', timeframe='2016-01-01 2020-12-31')
```

- The item `kw_list` is always required, and supplies the list of keywords we're querying as a parameter to the `build_payload` function. The maximum number of items allowed here is five.
- `geo` defines the geographic location your query is limited by. In this instance, the location is set to 'global' by using an empty string. You can narrow your payload by using the two-letter (_capitilized_) [country code](https://www.iban.com/country-codes).
- `timeframe` is set to `'today 5-y'` by _default_, which means from five years back until today's date. We want coherent results with full years, so we explicitly set the timeframe to `'2016-01-01 2020-12-31'`.

Let's retrieve the 'interest over time' data with the following command:

```
time_interest = pytrends.interest_over_time()
del time_interest['isPartial']
```

The `interest_over_time()` method returns data as a pandas dataframe. We can delete the `isPartial` (boolean) column from our dataframe as it signifies whether or not the data is complete and we won't need to plot this. You can run `time_interest.head()` to see what kind of data we're working with.

You can directly plot this data using `time_interest.plot()`, returning a graph similar to what you see on the Google Trends website. However, that is not our goal in this article, so let's continue and customise our plot.

Enter the following code:

```
time_interest = time_interest.groupby(pd.Grouper(freq='Y')).sum()

ax = time_interest.plot(kind='bar', xlabel='Year', ylabel="Interest", figsize=(10, 5))
ax.set_xticklabels([pandas_datetime.strftime("%Y") for pandas_datetime in time_interest.index])
ax.set_title('Interest Over Time', fontsize=20)
ax.xaxis.set_ticks_position('none')
```

- As we are aiming to plot a bar chart, we use `groupby` to group the data according to a particular parameter specified by `pd.Grouper`. In our example, we have chosen to group the data according to years (`freq='Y'`). This function operates on the index of our dataframe, which in this case is the dates. We add `sum()` at the end of the line to ensure that the data is aggregated and visualized on one plot instead of separate plots, each with their own years.
- We plot our `time_interest` graph as a bar chart with `kind='bar'`(pandas' `.plot()` method and it's arguments are explained [here](https://pandas.pydata.org/pandas-docs/version/0.15/generated/pandas.DataFrame.plot.html)). Set the figsize to `(10, 5)` (width x height in inches).
- Use `ax.set_xticklabels([pandas_datetime.strftime("%Y") for pandas_datetime in time_interest.index])` in list comprehension to get only the years from our dataframe as labels on the x-axis. Alternatively, you could just as well use `['2016','2017','2018','2019',..]`. If these labels aren't specified, pandas will include months, days and times in our graph, which we don't need.
- Finally, we set the title for our graph using `set_title` and use `ax.xaxis.set_ticks_position('none')` to remove the xticks report we would otherwise get on the graph.

Your graph should look something like this:
![](Screenshots/Interest_BarChart.svg)

Save your notebook file before continuing.

## GitHub

With our first graph done, let's go ahead and check our notebook into GitHub for safe storage.

Push your notebook onto a new repo in GitHub using the standard way through your terminal:

```
git init
git add -A
git commit -m "first commit"
git branch -M main
git remote add origin <Your Github repo>
git push -u origin main
```

## Creating a pie chart: Interest by Region

With our notebook safely checked in, let's create another graph below our first one. This time we'll create a pie chart showing the regions with the highest interest in a topic.

Type in the following code:

```
kw_list = ["stack overflow"]
pytrends.build_payload(kw_list)

regions = pytrends.interest_by_region()
regions = regions.sort_values(kw_list[0], ascending=False)[:15]
regions.head()
```

- As previously, we start by defining a keyword list and building a payload from it. This time, however, we'll use one query to visualize one pie chart. We'll use the keyword "stack overflow".
- We call `pytrends.interest_by_region()` to get our regional data in a pandas dataframe.
- We sort the data into descending order by specifying `kw_list[0]` and `ascending=False`. In this instance, we limit the payload to the top 15 regions interested in our specified topic: `[:15]`.
- You can run `regions.head()` to see the data we're working with (don't forget to rerun your notebook from the top in case you closed it earlier).

Now that we have our data ready, let's plot it using the following code:

```
ax = regions.plot(kind = 'pie', figsize=(10, 10), subplots=True, legend=False)
ax = ax.flatten()[0]
ax.set_title('Interest by Region', fontsize=20)
ax.xaxis.set_ticks_position('none')
```

You'll notice the code is very similar to what we used to plot the bar chart, the major differences being:

- `kind = 'pie'` to specify we want a pie chart.
- `subplots=True` enables us to plot the chart without having a y-column in our data.
- `legend=False` removes the extra legend, as the pie slices are already named by region.
- The data returns as a two-dimensional np.array. We use `ax.flatten()[0]` to collapse the np.array into one dimension so we can further process it as a subplot. It's not possible to set the title without first flattening.

You should have a graph similar to this, which provides an interesting take on the regional interest in a topic:
![](Screenshots/RegionInterest_PieChart.svg)

## GitHub diffs

With another graph done, let's save our notebook and commit it to the GitHub repo, pushing the changes we've added:

```
git commit -a -m "Added regional interest pie chart"
git push
```

Now we can review the changes on GitHub's website.

On your repo, open the latest change we've made on the notebook and click on 'load diff'.
![](Screenshots/Capture.png)
_A lot_ of JSON pops up, showing the minor changes we've made. This is a headache. Fortunately, we have a handy tool to let us review the changes.

## ReviewNB

[ReviewNB](https://www.reviewnb.com/) is a GitHub app specifically dealing with visual diffs and comments on notebooks. Go ahead and create an account for free if you don't already have one.

After authorizing the app and allowing it repository access to our current repo, you should be redirected to the app's [home page](https://app.reviewnb.com/). From here, let's select our repo and then the latest commit underneath the `Commits` tab.

When you open the notebook file, you should see something like this:
![](Screenshots/Capture1.png)

This is _not_ a headache. Now we can see exactly what we have changed in our notebook.

## Creating a KDE plot: Interest Over Time

With our version control issue solved, let's create one final custom Google Trends graph. This one will be a Kernel Density Estimation plot derived from our first graph.

Before we begin, let's create and checkout another branch on the repo using:

```
git branch WIP
git checkout WIP
```

Now we'll use much of our first bar chart code with a little modification. This is what it should look like:

```
kw_list = ["Java", "Python", "Javascript"]
pytrends.build_payload(kw_list, timeframe='2016-01-01 2020-12-31', cat=0, gprop='')
interest = pytrends.interest_over_time()
interest.head()
```

You'll notice that we've added two new arguments to the `build_payload` function:
- We've set `cat=0` (all categories) as we don't want any categories to narrow the results.
- `gprop` specifies what Google property to filter to, such as `youtube`, `images`, `news`, etc. We've left it empty so it defaults to web searches.

Now we can visualize our density plot using the following code:

```
ax = interest.plot(kind='density', figsize=(10, 10))
ax.set_xticks([])

ax.set_title('KDE Plot (Density of interest over Time)', fontsize=20)
ax.set_xlabel('2016 - 2020', fontsize=12)
ax.set_ylabel('Interest density', fontsize=12)
ax.xaxis.set_ticks_position('none')
```

This time, we do not aggregate the data as we will need to use all the points on the density plot. We can leave the `isPartial` column in-place as it will not be plotted on this particular chart. A few other changes are:

- `kind='density'` to indicate we want a density plot. You could also use `'kde'`.
- We remove the x axis values using `set_xticks([])` as the values end up being rather hard to work with (especially when trying to correlate them with the years).
- Notice that we do not set the x and y labels within the `.plot()` method, but rather `ax.set_xlabel()` and `ax.set_ylabel()`. This is to show that you can use this alternative technique in case you run into issues where the x label does not show even though you have defined it inside `.plot()`.

You can then run the code and you will get something similar to this:
![](/Screenshots/Interest_Density.svg)

It is interesting to view the spike in JavaScript interest in this manner.

## Pull Requests

Let's create a pull request scenario for ourselves as this will help us familiarize the GitHub workflow.

Save your notebook and push it into the new branch we created using:

```
git commit -a -m "added a density plot"
git push -u origin WIP
```

We have successfully pushed the new plot to our secondary branch.

Open GitHub and you should see "WIP had recent pushes less than a minute ago". Click on the `Compare and pull request` button. The next screen will allow you to write a comment and create a pull request.

After you have created a pull request, the `Conversation` tab will open where you can have a discussion with collaborators about the pull request you just created. You will notice that the `review-notebook-app` bot has commented with a link to view our pull request on ReviewNB. Let's not revive the JSON dragon and click on that link.

The link takes us to the `Changes` tab on ReviewNB which we can also get to via the normal [app route]('https://app.reviewnb.com/') and then selecting the `Pull requests` tab. From there we can see the changes we've added and even add a comment on certain lines if we need to get clarification or provide feedback.

Let's post a comment on the new graph. I've posted "**Could you lower the graph height to 60% - 75% of what it is? There's too much open space**". We can see our comment on the `Discussion` tab, and if we head back to GitHub, we will see the same thing in `Conversation`.

Finally, we can resolve the conversation and merge our WIP branch to main.

#

Possible Issues:

- It is possible to be rate limited when making excessive requests with the Pytrends API. Have a look at the [Connect to Google](https://pypi.org/project/pytrends/) section of the Pytrends project page for a remedy.

Have a look at Matplotlib's [gallery](https://matplotlib.org/stable/gallery/index.html) for more plot examples. [Kaggle](https://www.kaggle.com/datasets) also offers numerous open datasets.
