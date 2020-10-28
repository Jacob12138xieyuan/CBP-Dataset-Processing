# CBP-Dataset-Processing

```
import re
import pandas as pd
from os import listdir

# return all filename with .csv
def find_csv_filenames(path_to_dir, suffix=".csv" ):
    filenames = listdir(path_to_dir)
    return [ filename for filename in filenames if filename.endswith( suffix ) ]

old_filenames = find_csv_filenames("./csv")
filenames = old_filenames[19:]+old_filenames[:19] # sort them into 86,87,88...17,18 order
# print(filenames)

# function process every file
def process(filename):
    year = filename[3:5] # get year from filename e.g.18
    print(year)
    fields = ["fipstate", "naics", "est"] # only select these three columns
    df = pd.read_csv("./csv/"+filename, skipinitialspace=True, usecols=fields) # read csv file
    df['year'] = year # add a column called 'year'   
    regex = "^\d{3,}" # define regular express to filter out digits less than 3
    filtered = df[df.naics.str.contains(regex)] # filter
    filtered["naics"] = filtered.naics.str[:3] # only select first three digits
    grouped = filtered.groupby(['year', 'fipstate', 'naics']).sum().reset_index() # group by state and sum up est for each state
    state_code = pd.read_excel('code.xlsx') # get fips code 
    merged = pd.merge(left=grouped, right=state_code, left_on='fipstate', right_on='FIPS') # join two table based on fips 
    merged.drop(['fipstate', 'FIPS'],axis=1, inplace=True) # delete these columns since not interested
    merged = merged[['year', 'Name', 'Postal Code', 'naics', 'est']] # rearrange column order
    return merged

result = pd.DataFrame([], columns=["year","Name","Postal Code","naics","est"]) # init dataframe
for filename in filenames: # iterate every file
    df = process(filename)
    result = result.append(df, ignore_index = True) # append each year result to final result
result.to_excel(r'output.xlsx', index = False) # export as excel file
```
