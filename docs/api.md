# API Reference

The REST APIs provide programmatic ways to submit new jobs and to download data from Helmholtz Munich Imputation Server. It identifies users using authentication tokens, responses are provided in JSON format.


## Authentication
Helmholtz Munich Imputation Server uses a token-based authentication. The token is required for all future interaction with the server. The token can be created and downloaded from your user profile (username -> Profile):


![Activate API](https://raw.githubusercontent.com/genepi/imputationserver-docker/master/images/api.png)

For security reasons, Api Tokens are valid for 30 days. You can check the status in the web interface.


## Job Submission for Whole Genome Imputation
The API allows to submit imputation jobs and to set several parameters.

### POST /jobs/submit/imputationserver@2.0.0 

The following parameters can be set:

| Parameter        | Values           | Default Value  |  Required  |
| ------------- |:-------------| :-----|---|
| files         | /path/to/file |  | **x** |
| mode          | `qconly`<br> `phasing` <br> `imputation`     | `imputation`   | |
| password      | user-defined password      |  auto generated and send by mail  | |
| refpanel      | `1000g-phase-3-v5`   <br>  `hapmap-2` <br>  | - | **x** |
| phasing     | `eagle`<br> `no_phasing`      |  `eagle`  | |
| population  | `eur`<br> `afr`<br> `asn`<br> `amr`<br> `sas`<br> `eas`<br> `AA`<br> `mixed` <br> `all`   |  -  | **x** |
| build       | `hg19`<br> `hg38` | `hg19`  | |
| r2Filter    | `0` <br> `0.001` <br> `0.1` <br> `0.2` <br> `0.3` | `0`  | |





### Examples: curl

#### Submit a single file

To submit a job please change `/path-to/file.vcf.gz` to a valid vcf file and update `TOKEN` with your API Token:


Command:

```sh
TOKEN="YOUR-API-TOKEN";

curl https://imputationserver.helmholtz-munich.de/api/v2/jobs/submit/imputationserver@2.0.0 \
  -H "X-Auth-Token: $TOKEN" \
  -F "files=@/path-to/file.vcf.gz" \
  -F "refpanel=1000g-phase-3-v5" \
  -F "population=eur"
```

Response:

```json

{
"success":true,
"id":"job-20240425-001551-097",
"message":"Your job was successfully added to the job queue."
}
```


#### Submit multiple files

Submits multiple vcf files and impute against 1000 Genomes Phase 3 reference panel.

Command:

```sh
TOKEN="YOUR-API-TOKEN";

curl https://imputationserver.helmholtz-munich.de/api/v2/jobs/submit/imputationserver@2.0.0 \
  -H "X-Auth-Token: $TOKEN" \
  -F "files=@/path-to/file1.vcf.gz" \
  -F "files=@/path-to/file2.vcf.gz" \
  -F "refpanel=1000g-phase-3-v5" \
  -F "population=eur"
```

Response:

```json
{
  "id":"job-20120504-155023",
  "message":"Your job was successfully added to the job queue.",
  "success":true
}
```






### Examples: Python

#### Submit single vcf file
```python
import requests

url = 'https://imputationserver.helmholtz-munich.de/api/v2'
token = 'YOUR-API-TOKEN'

data = {
    'job-name': "test_api_token",
    'refpanel': '1000g-phase-3-v5',
    'build': 'hg19',
    'population': 'eur'
}

headers = {'X-Auth-Token' : token}

# submit new job
vcf = '/path/to/genome.vcf.gz';
files = {'files' : open(vcf, 'rb')}
r = requests.post(url + '/jobs/submit/imputationserver@2.0.0',
                  files=files, data=data, headers=headers)

if r.status_code != 200:
  print(r.json()['message'])
  raise Exception('POST /jobs/submit/imputationserver@2.0.0 {}'.format(r.status_code))

# print response and job id
print(r.json()['message'])
print(r.json()['id'])
```


#### Submit multiple vcf files

```python
import requests

url = 'https://imputationserver.helmholtz-munich.de/api/v2/'
token = 'YOUR-API-TOKEN'

data = {
    'job-name': "test_api_token",
    'refpanel': '1000g-phase-3-v5',
    'build': 'hg19',
    'population': 'eur'
}

headers = {'X-Auth-Token' : token}

# submit new job
vcf = '/path/to/file1.vcf.gz';
vcf1 = '/path/to/file2.vcf.gz';
files = [('files', open(vcf, 'rb')), ('files', open(vcf1, 'rb'))]
r = requests.post(url + 'jobs/submit/imputationserver@2.0.0',
                  files=files, data=data, headers=headers)

if r.status_code != 200:
  print(r.json()['message'])
  raise Exception('POST /jobs/submit/imputationserver@2.0.0 {}'.format(r.status_code))

# print message
print(r.json()['message'])
print(r.json()['id'])
```


## List all jobs
All running jobs can be returned as JSON objects at once.
### GET /jobs

### Examples: curl

Command:

```sh
TOKEN="YOUR-API-TOKEN";

curl -H "X-Auth-Token: $TOKEN" https://imputationserver.helmholtz-munich.de/api/v2/jobs
```


```json
{
    "count":1,
    "page":1,
    "pages":[1],
    "data": [{
         "app":null,
         "application":"Genotype Imputation (Minimac4), 2.0.0 2.0.0",
         "canceld":false,
         "complete":true,
         "currentTime":1714036281474,
         "endTime":1714034450468,
         "finishedOn":1714034451414,
         "id":"job-20240425-103410-874",
         "name":"test_api_token",
         "positionInQueue":-1,
         "priority":0,
         "progress":-1,
         "setupEndTime":1714034091801,
         "setupRunning":false,
         "setupStartTime":1714034051089,
         "startTime":1714034095135,
         "state":5,
         "submittedOn":1714034051037,
         "workspaceSize":""
  }]
}
```

### Example: Python

```python
import json
import requests

# imputation server url
url = 'https://imputationserver.helmholtz-munich.de/api/v2'
token = 'YOUR-API-TOKEN'

# add token to header (see authentication)
headers = {'X-Auth-Token' : token}

# get all jobs
r = requests.get(url + "/jobs", headers=headers)
if r.status_code != 200:
    raise Exception('GET /jobs/ {}'.format(r.status_code))

# print all jobs
jobs = r.json()
for job in jobs['data']:
    print(json.dumps(job, indent=4))
```


```json

{
    "app": null,
    "application": "Genotype Imputation (Minimac4), 1.6.8 1.6.8",
    "canceld": false,
    "complete": true,
    "currentTime": 1714040218658,
    "endTime": 0,
    "finishedOn": 1714038005353,
    "id": "job-20240425-113941-495",
    "name": "test_api_token1",
    "positionInQueue": -1,
    "priority": 0,
    "progress": -1,
    "setupEndTime": 1714038002250,
    "setupRunning": false,
    "setupStartTime": 1714037981736,
    "startTime": 0,
    "state": 5,
    "submittedOn": 1714037981658,
    "workspaceSize": ""
}
```
## Monitor Job Status

### /jobs/{id}/status

### Example: curl

Command:

```sh
TOKEN="YOUR-API-TOKEN"

curl -H "X-Auth-Token: $TOKEN" https://imputationserver.helmholtz-munich.de/api/v2/jobs/job-20240425-090137-703/status

```

Response:

```json

{
    "app": null,
    "application": "Genotype Imputation (Minimac4), 2.0.0 2.0.0",
    "applicationId": "imputationserver@2.0.0",
    "canceld": false,
    "complete": true,
    "currentTime": 1714040634188,
    "deletedOn": -1,
    "endTime": 1714034450468,
    "finishedOn": 1714034451414,
    "id": "job-20240425-103410-874",
    "logs": "",
    "name": "test_api_token",
    "outputParams": [],
    "positionInQueue": -1,
    "priority": 0,
    "progress": -1,
    "running": false,
    "setupEndTime": 1714034091801,
    "setupRunning": false,
    "setupStartTime": 1714034051089,
    "startTime": 1714034095135,
    "state": 5,
    "steps": [],
    "submittedOn": 1714034051037,
    "workspaceSize": ""
}


```

### Example: Python

```python
import requests
import json

# imputation server url
url = 'https://imputationserver.helmholtz-munich.de/api/v2'
token = "YOUR-API-TOKEN"
job_id = 'job-20240425-103410-874'

# add token to header (see authentication)
headers = {'X-Auth-Token' : token}

# get all jobs
r = requests.get(url + "/jobs/{}/status".format(job_id), headers=headers)
if r.status_code != 200:
    raise Exception('GET /jobs/{}/status {}'.format(job_id, r.status_code))

print(json.dumps(r.json(), indent=4))

```

```json
{
    "app": null,
    "application": "Genotype Imputation (Minimac4), 2.0.0 2.0.0",
    "applicationId": "imputationserver@2.0.0",
    "canceld": false,
    "complete": true,
    "currentTime": 1714040634188,
    "deletedOn": -1,
    "endTime": 1714034450468,
    "finishedOn": 1714034451414,
    "id": "job-20240425-103410-874",
    "logs": "",
    "name": "test_api_token",
    "outputParams": [],
    "positionInQueue": -1,
    "priority": 0,
    "progress": -1,
    "running": false,
    "setupEndTime": 1714034091801,
    "setupRunning": false,
    "setupStartTime": 1714034051089,
    "startTime": 1714034095135,
    "state": 5,
    "steps": [],
    "submittedOn": 1714034051037,
    "workspaceSize": ""
}
```



## Monitor Job Details

### /jobs/{id}

### Example: curl

```sh
TOKEN="YOUR-API-TOKEN"

curl -H "X-Auth-Token: $TOKEN" https://imputationserver.helmholtz-munich.de/api/v2/jobs/job-20240425-090137-703/
```
### Example: Python

```python
# imputation server url
url = 'https://imputationserver.helmholtz-munich.de/api/v2'
token = "YOUR-API-TOKEN"
job_id = 'job-20240425-103410-874'

# add token to header (see authentication)
headers = {'X-Auth-Token' : token}

# get all jobs
r = requests.get(url + "/jobs/{}".format(job_id), headers=headers)
if r.status_code != 200:
    raise Exception('GET /jobs/{} {}'.format(job_id, r.status_code))

print(json.dumps(r.json(), indent=4))

```

## Cancel Job

### /jobs/{id}/cancel

### Example: curl

```sh
TOKEN="YOUR-API-TOKEN"

curl -H "X-Auth-Token: $TOKEN" https://imputationserver.helmholtz-munich.de/api/v2/jobs/job-20240425-090137-703/cancel
```

### Example: Python

```python
# imputation server url
url = 'https://imputationserver.helmholtz-munich.de/api/v2'
token = "YOUR-API-TOKEN"
job_id = 'job-20240425-103410-874'

# add token to header (see authentication)
headers = {'X-Auth-Token' : token}

# get all jobs
r = requests.get(url + "/jobs/{}/cancel".format(job_id), headers=headers)
if r.status_code != 200:
    raise Exception('GET /jobs/{}/cancel {}'.format(job_id, r.status_code))

print(json.dumps(r.json(), indent=4))
```




