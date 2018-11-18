## Tips Part Two

```
df = pd.concat([pd.read_csv(file) for file in glob.glob('./data/*')], ignore_index = True)
```

`df.dtypes`

`df.isna().sum()`

`df = df[df.column.notna()]`

`df['column'] = df['column'].fillna(0) or. fillna('missing')`

```
# take only the numbers in a string (and divide by 100 to get dollars instead of cents, for example)
df['column'] = df['column'].map(lambda string: float(''.join([char for char in string if char.isdigit()]))/100)
# another lambda function to use:
lambda string: string.replace('[', '<')
```

```
# ternary operator
df['skills'] = df['skills'].map(lambda sk_list: (sk_list, [])[sk_list == ['']])
```

`skills_df = df['skills'].str.join('|').str.get_dummies()`

`combined = pd.concat([skills_df, rate], axis = 1, sort = False)`

### Grouped Boxplot

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
