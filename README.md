# Reading NWB files

## Python libraries
To read NWB files using python you must have the pynwb package
See [here](https://pynwb.readthedocs.io/en/stable/install_users.html) for instructions on installing package

## Opening the File in python
To open the file in python you must first import NWBHDF5IO from the pynwb package
```python
from pynwb import NWBHDF5IO
```
---
Then do the following,
```python
with NWBHDF5IO('path/to/nwb/file.nwb', 'r') as io:
    nwbfile = io.read()
    print(nwbfile)
```

## Location of Raw Data
NWB files are basically stored like a list of dictionaries
With fields such as **acquisition**, **devices**, and **electrodes**

Acquisition holds all the raw data
The syntax to open acquisition is
```python
print(nwbfile.acquisition)
```
---
From here you can access all of the raw data.

There should be 12 datasets
```python
with NWBHDF5IO('/path/to/nwb/file.nwb', 'r') as io:
    nwbfile = io.read()
    print(list(nwbfile.acquisition.keys()))
```
`['ECG', 'EDA', 'EMG', 'ElectricalSeries', 'Eyetrack_Argus', 'Head_Location_Argus', 'Head_Rotation_Argus', 'Monitor_Eyetrack_Argus', 'Pupil_Diameters_Argus', 'RESPIRATION', 'StimLabels', 'allCSTdata']`

Within each of these datasets there are different attributes.

To get the raw data we use the `.data` attribute

For example, to access the EEG data, we can use this syntax,
```python
print(nwbfile.acquisition['ElectricalSeries'].data)
```
You might see this output,

`<HDF5 dataset "data": shape (397540, 64), type "<f4">`

This is the object that is loaded by pynwb that can be indexed.
To load the entire array into memory, just use `[:]` at the end

All raw data are stored as numpy arrays.

---
## Other Attributes
There are other attributes within each dataset
`.data` is just one of them
`.timestamps` gives the timestamp of each data point
For the EEG data specifically, `.electrodes[:]`can be used to display 
the entire table of electrodes detailing informations such as
the general location of each electrode, the coordinates of the electrode in mm,
and the impedance of each electrode for each run in kohms.

## CST data
Since all of the cst data is loaded together, you can use the following snippet of code to load it
into a pandas dataframe with the correct headers
```python
with NWBHDF5IO('/path/to/nwb/file.nwb', 'r') as io:
        nwbfile = io.read()
        cst = nwbfile.acquisition['allCSTdata'].data[:]
        times = nwbfile.acquisition['allCSTdata'].timestamps[:]
        headers = nwbfile.acquisition['allCSTdata'].description.split(',')
        units = nwbfile.acquisition['allCSTdata'].unit.split(',')

df = pd.DataFrame(cst, columns=headers)
df['times'] = times
print(df)
```

## pynwb Documentation
For more information the pynwb documentation can be found [here](https://pynwb.readthedocs.io/en/stable/index.html)

## NWB Documentation
For more information about the NWB file format, the documentation can be found [here](https://nwb-overview.readthedocs.io/en/latest/index.html)
