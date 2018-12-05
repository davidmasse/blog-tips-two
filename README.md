## Pandas Patterns for ETL and EDA

This post for my own reference continues the themes of and ![earlier blog post](https://medium.com/@davidmasse8/helpful-plotting-and-pandas-patterns-80fd82b2b88b).


### Uploading Multiple Files to One Dataframe

Reads and combines all the CSV files in the `/data` folder of the current directory:
```
import glob
df = pd.concat([pd.read_csv(file) for file in glob.glob('./data/*')], ignore_index = True)
```

The above tacks the rows from each CSV file onto the bottom of the dataframe, but they could also be combined (if they have the same number of rows) "horizontally" (i.e. by column): `combined = pd.concat([skills_df, rate], axis = 1, sort = False)`

### More Initial Checks

Missing values: Note that when using expressions like `df.isnull().sum()` to count missing values, `isnull()` is an alias for `isna()`, which could just as easily be used.  Both `NULL` and `NA` (different in R) in Pandas refer to Numpy's `NaN`.

`.notna()` to keep only rows where the column is not null: `df = df[df.column.notna()]`

`.fillna()` as way to interpret missing values in a way that makes sense for the DataFrame at hand:
`df['column'] = df['column'].fillna(X)`.  If the column has numeric values, `X` here might be zero if missing values should really mean "none."  If the column has categorical values, `X` could be `"other"`, a separate category.

### Cleaning and Transforming the DataFrame

Take only the numbers in a string (and divide by 100 to get dollars instead of cents, for example):
`df['column'] = df['column'].map(lambda string: float(''.join([char for char in string if char.isdigit()]))/100)`

Another lambda function to use: `lambda string: string.replace('[', '<')`

Ternary operator for lambda function: `df['skills'] = df['skills'].map(lambda sk_list: (sk_list, [])[sk_list == ['']])`


`skills_df = df['skills'].str.join('|').str.get_dummies()`



### Grouped Boxplot

Here's a way to make a grouped boxplot (on the same scale), with the width of the box proportional to the number of samples in that group:

```
# Numpy, not Pandas, so use .stack instead of .concat
for_box = np.stack((clust_assigned, y), axis = 1)
box_df = pd.DataFrame(for_box)
box_df.columns = ['cluster', 'rate']
dfg = box_df.groupby('cluster')
counts = [len(v) for k, v in dfg]
total = float(sum(counts))
cases = len(counts)
widths = [c/total for c in counts]  
cax = box_df.boxplot(column='rate', by='cluster', figsize=(15,10))
cax.set_xticklabels(['%s\n$n$=%d'%(k, len(v)) for k, v in dfg])
cax.set_ylim([0,200])
```

Rename a column with a dictionary:
`result.rename(columns={0: 'cluster'}, inplace=True)`

### ETL Vignette: Create a Dataframe from Variable-Length Data with Alternate Labels for the Same Concept/Column

If you are extracting values one by one from a website's code (or any data file that is flexibly organized), sometimes, for each record, there could be a variable number of values given (those omitted from the maximal set are presumed 'NA').  The labels for the values, which are harvested at the same time, are contained in each record in the same list items as the corresponding values.  However, one of the values (an integer) is sometimes labeled with the singular "Job" and other times with the plural "Jobs."

We first make the dictionary that will become a row in our dataframe.  We then make another dictionary (this could be defined globally or passed into the function) called `switcher` that translates both "Jobs" and "Job" to "jobs" along with the other possible labels.

```
def get_last_three():
    three = {'hours': 'NA',
            'jobs': 'NA',
            'earned': 'NA'}
    switcher = {'Total earned': 'earned',
               'Jobs': 'jobs',
               'Job': 'jobs',
               'Hours worked': 'hours'}
    # d is a Python list of HTML 'li' (list item) elements containing the values and labels we want
    d = driver.find_element_by_class_name('cfe-aggregates').find_element_by_tag_name('ul').find_elements_by_tag_name('li')
    # the first element was always the same in this example, and extracted elsewhere, so we start with the second
    for i d[1:]:
        # pick out the label
        label = i.find_element_by_class_name('text-muted').text
        # look up the label in the switcher dictionary to find a key for the dictionary called 'three',
        # pick out the value in the HTML, and pair it with the key just looked up:
        three[switcher[label]] = i.find_element_by_tag_name('h3').text
    return three
```
