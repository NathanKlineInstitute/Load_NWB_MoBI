# Reading NWB files

## Python libraries
To read NWB files using python you must have the pynwb package.

See [here](https://pynwb.readthedocs.io/en/stable/install_users.html) for instructions on installing package.

## Opening the File in python
To open the file in python you must first import NWBHDF5IO from the pynwb package.
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

## Structure of NWB File
NWB files are basically stored like a list of dictionaries with fields such as **acquisition**, **devices**, and **electrodes**.

---

### Acquisition
`nwbfile.acquisition`

Acquisition holds all the raw datasets.

Each dataset then has additional attributes.

Depending on the task, there are a varying amount of datasets

For example, there are 9 datasets for the sample cst files:
```python
with NWBHDF5IO('/path/to/nwb/file.nwb', 'r') as io:
    nwbfile = io.read()
    print(list(nwbfile.acquisition.keys()))
```
`['ElectricalSeries', 'Eyetrack_Argus', 'Head_Location_Argus', 'Head_Rotation_Argus', 'Monitor_Eyetrack_Argus', 'Pupil_Diameters_Argus', 'StimLabels', 'allCSTdata', 'allOpenSignalsData']`

---

### ElectricalSeries
`nwbfile.acquisition['ElectricialSeries']`

The 'ElectricalSeries' dataset holds the information about the eeg data in microvolts.

The relavent attributes this dataset has are:
- `.data` this holds all the raw data
- `.timestamps` this holds all the timestamps
- `.description` this holds the column headers as a string. Split string by ',' to get as a list for use in pandas dataframe
- `.electrodes` Stores table for detailed information about electrodes such as general location, coordinates, and impedances.

### Eyetrack_Argus
`nwbfile.acquisition['Eyetrack_Argus'].spatial_series['Eyetrack_Argus']`

The 'Eyetrack_Argus' dataset holds information about the data collected from the argus eyetracker, specifically the position of the eye as it moves.

The relavent attributes this dataset has are:
- `.data` this holds all the raw data
- `.timestamps` this holds all the timestamps
- `.description` this holds the column headers as a string. Split string by ',' to get as a list for use in pandas dataframe

### Monitor_Eyetrack_Argus
`nwbfile.acquisition['Monitor_Eyetrack_Argus'].spatial_series['Monitor_Eyetrack_Argus']`

The 'Monitor_Eyetrack_Argus' dataset holds information about the data collected from the argus eyetracker, specifically where the eye is looking at the monitor.

The relavent attributes this dataset has are:
- `.data` this holds all the raw data
- `.timestamps` this holds all the timestamps
- `.description` this holds the column headers as a string. Split string by ',' to get as a list for use in pandas dataframe

### Head_Location_Argus
`nwbfile.acquisition['Head_Location_Argus'].spatial_series['Head_Location_Argus']`

The 'Head_Location_Argus' dataset holds information about the data collected from the argus eyetracker, specifically the coordinates in cm of the head as it moves in space.
`0,0,0` is located at the tracker. +X is distance from monitor, +Y is Right, +Z is Down'

The relavent attributes this dataset has are:
- `.data` this holds all the raw data
- `.timestamps` this holds all the timestamps
- `.description` this holds the column headers as a string. Split string by ',' to get as a list for use in pandas dataframe

### Head_Rotation_Argus
`nwbfile.acquisition['Head_Rotation_Argus']`

The 'Head_Rotation_Argus' dataset holds information about the data collected from the argus eyetracker, specifically the rotation of the head as it moves in space.
`-180,0,0` is the subject looking straight ahead at the monitor.

The relavent attributes this dataset has are:
- `.data` this holds all the raw data
- `.timestamps` this holds all the timestamps
- `.description` this holds the column headers as a string. Split string by ',' to get as a list for use in pandas dataframe

### Pupil_Diameters_Argus
`nwbfile.acquisition['Pupil_Diameters_Argus']`

The 'Pupil_Diameters_Argus' dataset holds information about the data collected from the argus eyetracker, specifically the diameter of the pupil

The relavent attributes this dataset has are:
- `.data` this holds all the raw data
- `.timestamps` this holds all the timestamps
- `.description` this holds the column headers as a string. Split string by ',' to get as a list for use in pandas dataframe

### Stimlabels
`nwbfile.acquisition['StimLabels']`

The 'StimLabels' dataset holds information about StimLabels

The relavent attributes this dataset has are:
- `.data` this holds all the raw data
- `.timestamps` this holds all the timestamps

### allCSTdata
`nwbfile.acquisition['allCSTdata']`

The 'allCSTdata' dataset holds information about CST data

The relavent attributes this dataset has are:
- `.data` this holds all the raw data
- `.timestamps` this holds all the timestamps
- `.description` this holds the column headers as a string. Split string by ',' to get as a list for use in pandas dataframe
- `.units` this holds the units for each column as a string

### allOpenSignalsData
`nwbfile.acquisition['allOpenSignalsData']`

The 'allOpenSignalsData' dataset holds information about general biological data

The relavent attributes this dataset has are:
- `.data` this holds all the raw data
- `.timestamps` this holds all the timestamps
- `.description` this holds the column headers as a string. Split string by ',' to get as a list for use in pandas dataframe
- `.units` this holds the units for each column as a string

### Mindlogger
`nwbfile.acquisition['Mindlogger']`

The 'Mindlogger' dataset holds information about MindLogger data

The relavent attributes this dataset has are:
- `.data` this holds all the raw data
- `.timestamps` this holds all the timestamps
- `.description` this holds the column headers as a string. Split string by ',' to get as a list for use in pandas dataframe

---

## Loading Data

You might see this output when loading `.data`,

`<HDF5 dataset "data": shape (397540, 64), type "<f4">`

This is the object that is loaded by pynwb to be indexed.
To load the entire array into memory, just use `[:]` at the end.

All raw data are stored as numpy arrays.

---

## Creating pandas dataframe from data

Since some data is loaded together with column headers in the description, you can create a pandas dataframe with everything provided.
For example, here is a snippet of code to turn 'allCSTdata' into a pandas dataframe:
```python
from pynwb import NWBHDF5IO
import pandas as pd

with NWBHDF5IO('/path/to/nwb/file.nwb', 'r') as io:
    nwbfile = io.read()
    # Data Array
    cst = nwbfile.acquisition['allCSTdata'].data[:]
    # Timestamps for each data point
    times = nwbfile.acquisition['allCSTdata'].timestamps[:]
    # Description for cst data holds all the headers
    headers = nwbfile.acquisition['allCSTdata'].description.split(',')
    # The units for each column, not used in dataframe but can be called for later use
    units = nwbfile.acquisition['allCSTdata'].unit.split(',')

# Creating Dataframe
df = pd.DataFrame(cst, columns=headers)
# Adding timestamps as a column
df['times'] = times
print(df)
```

## pynwb Documentation
For more information the pynwb documentation can be found [here](https://pynwb.readthedocs.io/en/stable/index.html)

## NWB Documentation
For more information about the NWB file format, the documentation can be found [here](https://nwb-overview.readthedocs.io/en/latest/index.html)
