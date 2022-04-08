# KkOrthanc

Python library that wraps the Orthanc REST API.

## Installation

```
$ pip install KkOrthanc
```

## Example of usage

Be sure that Orthanc is running. The default URL (if running locally) is `http://localhost:8042`. 

### Get all the patients, studies, series and instances identifiers:

```python
from orthanc import Orthanc
orthanc = Orthanc("http://localhost:8042/")

patients_identifiers = orthanc.get_patients()
study_identifiers = orthanc.get_studies()
series_identifiers = orthanc.get_series()
instance_identifiers = orthanc.get_instances()
```

### Get specified patient, study, series, instance information:

```python
from orthanc import Orthanc
orthanc = Orthanc("http://localhost:8042/")

# To get patients identifiers
patients_identifiers = orthanc.get_patients()

# To get patient's studies identifier and main information
for patient_identifier in patients_identifiers:
    patient_information = orthanc.get_patients_info(patient_identifier)
    study_identifiers = patient_information['Studies']   
    # To get study's series identifier and main information
    for study_identifier in study_identifiers:
        study_information = orthanc.get_studies_info(study_identifier)	
        series_identifiers = study_information['Series']
        # To get series' instances identifier and main information
        for series_identifier in series_identifiers:
            series_information = orthanc.get_series_info(series_identifier)	
            instance_identifiers = series_information['Instances']
            # To get instances main information
            for instance_identifier in instance_identifiers:
                instance_information = orthanc.get_instances_info(instance_identifier)
```

### Delete specified study

```python
from orthanc import Orthanc
orthanc = Orthanc("http://localhost:8042/")

studies_identifiers = orthanc.get_studies()
print(studies_identifiers)
>>> ['c4bd9044-6b9654bd-43d61bca-2c58acd5-458c7854']
orthanc.delete_study('c4bd9044-6b9654bd-43d61bca-2c58acd5-458c7854')

studies_identifiers = orthanc.get_studies()
print(studies_identifiers)
>>> []
```

### Post specified instance

```python
from orthanc import Orthanc
orthanc = Orthanc("http://localhost:8042/")

with open('/Users/apple/Desktop/Instance.dcm', 'rb') as file:
    orthanc.post_instances(file.read())
```

### Delete specified instance

```python
from orthanc import Orthanc
orthanc = Orthanc("http://localhost:8042/")

instances_identifiers = orthanc.get_instances()
print(instances_identifiers)
>>> ['5dbdad22-d21b8e67-83358204-fc591c97-55571dda']
orthanc.delete_instance('5dbdad22-d21b8e67-83358204-fc591c97-55571dda')

instances_identifiers = orthanc.get_instances()
print(instances_identifiers)
>>> []
```

