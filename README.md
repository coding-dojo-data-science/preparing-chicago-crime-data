# preparing-chicago-crime-data
 

- Updated 11/15/22

>- This notebook will process a "Crimes - 2001 to Preset.csv" crime file in your Downloads folder and save it as several smaller .csv.gz's in a new "Data/Chicago/" folder inside this notebook's folder/repo.

## INSTRUCTIONS

- 1) Go to the Chicago Data Portal's page for ["Crimes - 2001 to Preset"](https://data.cityofchicago.org/Public-Safety/Crimes-2001-to-Present/ijzp-q8t2).

- 2) Click on the Export button on the top right and select CSV. 
    - Save the file to your Downloads folder instead of your repository. **The file is too big for a repository.**
    
    
    
- 3) Wait for the full file to download. 
    - It is very large (over >1GB and may take several minutes to fully download.)
    
    
- 4) Once the download is complete, change `RAW_FILE` variable below to match the filepath to the downloaded file.

## 🚨 Set the correct `RAW_FILE` path

- The cell below will attempt to check your Downloads folder for any file with a name that contains "Crimes_-_2001_to_Present".
    - If you know the file path already, you can skip the next cell and just manually set the RAW_FILE variable in the following code cell.


```python
## Run the cell below to attempt to programmatically find your crime file
import os,glob

## Getting the home folder from environment variables
home_folder = os.environ['HOME']
# print("- Your Home Folder is: " + home_folder)

## Check for downloads folder
if 'Downloads' in os.listdir(home_folder):
    
    
    # Print the Downloads folder path
    dl_folder = os.path.abspath(os.path.join(home_folder,'Downloads'))
    print(f"- Your Downloads folder is '{dl_folder}/'\n")
    
    ## checking for crime files using glob
    crime_files = sorted(glob.glob(dl_folder+'/**/Crimes_-_2001_to_Present*',recursive=True))
    
    # If more than 
    if len(crime_files)==1:
        RAW_FILE = crime_files[0]
        
    elif len(crime_files)>1:
        print('[i] The following files were found:')
        
        for i, fname in enumerate(crime_files):
            print(f"\t- File {i}) '{fname}'")
        print(f'\n- Please fill in the RAW_FILE variable in the code cell below with the correct filepath.')

else:
    print(f'[!] Could not programmatically find your downloads folder.')
    print('- Try using Finder (on Mac) or File Explorer (Windows) to navigate to your Downloads folder.')

```


```python
## MAKE SURE TO CHANGE THIS VARIABLE TO MATCH YOUR LOCAL FILE NAME
RAW_FILE = "YOUR FILEPATH HERE!"
```

# 🔄 Full Workflow

- Now that your RAW_FILE variable is set either:
    - On the toolbar, click on the Kernel menu > "Restart and Run All".
    - OR click on this cell first, then on the toolbar click on the "Cell" menu > "Run All Below"


```python
import pandas as pd
pd.set_option('display.max_columns', 100)

chicago_full = pd.read_csv(RAW_FILE)
chicago_full
```


```python
# this cell can take a minute + to run
chicago_full['Datetime'] = pd.to_datetime(chicago_full['Date'])
chicago_full = chicago_full.sort_values('Datetime')
chicago_full = chicago_full.set_index('Datetime')
chicago_full
```


```python
(chicago_full.isna().sum()/len(chicago_full)).round(2)
```

## Separate the Full Dataset by Years

### Creating Bins for Files


```python
# save the years for every crime
years = chicago_full.index.year
years
```


```python
# calculate 5-year bins
year_bins = (chicago_full.index.year-2000)//5
year_bins
```


```python
# Fill in a new file_bin column
chicago_full['file_bin'] = year_bins
chicago_full['file_bin'].value_counts(dropna=False)
```


```python
## still too big, drop some columns OR SEP INTO 2 FILES (one for essential info, one for other)
drop_cols = ["X Coordinate","Y Coordinate",#"Latitude","Longitude"'Updated On',
            "Community Area","FBI Code"]#,"IUCR",]
```


```python
chicago_full
```


```python
## columns to keep
keep_cols = list(chicago_full.drop(columns=['file_bin',*drop_cols]).columns)
keep_cols
```


```python
# unique # of year bins
file_bins = chicago_full['file_bin'].unique()
file_bins
```


```python
from tqdm.notebook import tqdm
import os
```


```python
## set save location 
folder = 'Data/Chicago/'
os.makedirs(folder, exist_ok=True)


for curr_bin in tqdm(file_bins):
    ## save temp slices of dfs to save.
    temp_df = chicago_full.loc[ chicago_full['file_bin']==curr_bin,
                                keep_cols].sort_index()

    ## get years for filename
    start = temp_df.index.year.min()
    end = temp_df.index.year.max()
    fname_temp = f"{folder}Chicago-Crime_{start}-{end}.csv.gz"
    temp_df.to_csv(fname_temp,compression='gzip',index=False)

    print(fname_temp)
```


```python
import glob
saved_files = sorted(glob.glob(folder+'*.*csv.gz'))
saved_files
```


```python
## create a README.txt for the zip files
readme = """Source URL: 
- https://data.cityofchicago.org/Public-Safety/Crimes-2001-to-Present/ijzp-q8t2
- Filtered for years 2000-Present.

Downloaded 07/18/2022
- Files are split into ~5 years per file.

EXAMPLE USAGE:
>> import glob
>> import pandas as pd
>> folder = "Data/Chicago/"
>> crime_files = sorted(glob.glob(folder+"*.csv.gz"))
>> df = pd.concat([pd.read_csv(f) for f in crime_files])
"""
print(readme)


with open(f"{folder}README.txt",'w') as f:
    f.write(readme)
```

## Confirmation

- Follow the example usage above to test if your files were created successfully.


```python
import glob
import pandas as pd
crime_files = sorted(glob.glob(folder+"*.csv.gz"))
df = pd.concat([pd.read_csv(f) for f in crime_files])
df
```
