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
Then to read the file, use the following code snippet,
```python
with NWBHDF5IO('path/to/nwb/file.nwb', 'r') as io:
    nwbfile = io.read()
    print(nwbfile)
```

## Structure of NWB File
NWB files are basically stored like a list of dictionaries with fields such as **acquisition**, **devices**, and **electrodes**.

```
Fields:
  acquisition: {
    Argus_Eye_Tracker <class 'pynwb.base.TimeSeries'>,
    BrainVision RDA <class 'pynwb.ecephys.ElectricalSeries'>,
    MindLogger <class 'pynwb.base.TimeSeries'>,
    OpenSignals <class 'pynwb.base.TimeSeries'>,
    StimLabels <class 'pynwb.base.TimeSeries'>
  }
  devices: {
    eeg_cap <class 'pynwb.device.Device'>
  }
  electrode_groups: {
    Brainvision_RDA_Electrodes <class 'pynwb.ecephys.ElectrodeGroup'>
  }
  electrodes: electrodes <class 'pynwb.ecephys.ElectrodesTable'>
  file_create_date: [datetime.datetime(2026, 1, 1, 11, 59, 59, 953265, tzinfo=tzoffset(None, -14400))]
  identifier: 129b25a4-db61-713e-8f4f-bd5be26410dd
  institution: RFMH
  lab: C-BIN
  session_description: LSL XDF Conversion
  session_start_time: 1970-01-01 00:00:00+00:00
  subject: subject pynwb.file.Subject at 0x140042508135648
Fields:
  age: P0D/
  age__reference: birth
  description: none
  sex: U
  species: HomoSapiens
  subject_id: A00012345

  timestamps_reference_time: 1970-01-01 00:00:00+00:00
```

---

### Acquisition
`nwbfile.acquisition`

Acquisition holds all the raw datasets.

Each dataset then has additional attributes.

Depending on the task, there are a varying amount of datasets

For example, there are 5 datasets for this sample spirals files:
```python
with NWBHDF5IO('/path/to/nwb/file.nwb', 'r') as io:
    nwbfile = io.read()
    print(list(nwbfile.acquisition.keys()))
```
`['Argus_Eye_Tracker', 'BrainVision RDA', 'MindLogger', 'OpenSignals', 'StimLabels']`

---

#### Accessing parts of datasets
`nwbfile.acquisition['BrainVision RDA']`

Each dataset holds information such as the raw data, timestamps, description, etc.

The relevant attributes datasets have are:
- `.data` this holds all the raw data
- `.timestamps` this holds all the timestamps
- `.description` this holds the column headers as a string. Split string by ',' to get as a list for use in pandas dataframe
- `.electrodes` (ONLY IN EEG DATASET) Stores table for detailed information about electrodes such as general location, coordinates, and impedances.

---

#### Loading Data

You might see this output when loading `.data`,

`<HDF5 dataset "data": shape (397540, 64), type "<f4">`

This is the object that is loaded by pynwb to be indexed.
To load the entire array into memory, just use `[:]` at the end.

All raw data are stored as numpy arrays.

---

#### Creating pandas dataframe from data

Since some data is loaded together with column headers in the description, you can create a pandas dataframe with everything provided.
For example, here is a snippet of code to turn 'allCSTdata' into a pandas dataframe:
```python
from pynwb import NWBHDF5IO
import pandas as pd

with NWBHDF5IO('/path/to/nwb/file.nwb', 'r') as io:
    nwbfile = io.read()
    streams = list(nwbfile.acquisition.keys())
    # Data Array
    if 'Argus_Eye_Tracker' in streams:
        argus_stream = nwbfile.acquisition['Argus_Eye_Tracker']
        argus_data = argus_stream.data[:]
        argus_timestamps = argus_stream.timestamps[:]
        argus_headers = argus_stream.description.split(',')

        df = pd.DataFrame(argus_data, columns=argus_headers)
        df['times'] = argus_timestamps
        print(df)

```

## pynwb Documentation
For more information the pynwb documentation can be found [here](https://pynwb.readthedocs.io/en/stable/index.html)

## NWB Documentation
For more information about the NWB file format, the documentation can be found [here](https://nwb-overview.readthedocs.io/en/latest/index.html)
