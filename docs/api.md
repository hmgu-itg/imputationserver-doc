# API Reference

The REST APIs provide programmatic ways to submit new jobs and to download data from Munich Imputation Server. It identifies users using authentication tokens, responses are provided in JSON format.


## Authentication
Munich Imputation Server uses a token-based authentication. The token is required for all future interaction with the server. The token can be created and downloaded from your user profile (username -> Profile):

![Activate API](https://raw.githubusercontent.com/genepi/imputationserver-docker/master/images/api.png)

For security reasons, Api Tokens are valid for 30 days. You can check the status in the web interface.


## Job Submission for Whole Genome Imputation
The API allows to submit imputation jobs and to set several parameters. For HLA imputation, please see below. 

### POST /jobs/submit/minimac4

The following parameters can be set:

| Parameter        | Values           | Default Value  |  Required  |
| ------------- |:-------------| :-----|---|
| files         | /path/to/file |  | **x** |
| mode          | `qconly`<br> `phasing` <br> `imputation`     | `imputation`   | |
| password      | user-defined password      |  auto generated and send by mail  | |
| files-source  | `file-upload`<br> `sftp`<br> `http`     |  `file-upload`  | |
| refpanel      | `hrc-r1.1` <br> `1000g-phase-3-v5` <br>  `gasp-v2` <br>  `genome-asia-panel`  <br> `1000g-phase-1` <br> `cappa` <br> `hapmap-2` | - | **x** |
| phasing     | `eagle`<br> `no_phasing`      |  `eagle`  | |
| population  | `eur`<br> `afr`<br> `asn`<br> `amr`<br> `sas`<br> `eas`<br> `AA`<br> `mixed` <br> `all`   |  -  | **x** |
| build       | `hg19`<br> `hg38` | `hg19`  | |
| r2Filter    | `0` <br> `0.001` <br> `0.1` <br> `0.2` <br> `0.3` | `0`  | |


## List all jobs
All running jobs can be returned as JSON objects at once.
### GET /jobs

### Examples: curl

Command:

```sh
TOKEN="YOUR-API-TOKEN";

curl -H "X-Auth-Token: $TOKEN" https://imputationserver.sph.umich.edu/api/v2/jobs
```

Response:

```json
[
  {
    "applicationId":"minimac",
    "executionTime":0,
    "id":"job-20160504-155023",
    "name":"job-20160504-155023",
    "positionInQueue":0,
    "running":false,
    "state":5
  },{
    "applicationId":"minimac",
    "executionTime":0,
    "id":"job-20160420-145809",
    "name":"job-20160420-145809",
    "positionInQueue":0,
    "running":false,
    "state":5
  },{
    "applicationId":"minimac",
    "executionTime":0,
    "id":"job-20160420-145756",
    "name":"job-20160420-145756",
    "positionInQueue":0,
    "running":false,
    "state":5
  }
]
```

### Example: Python

```python
import requests
import json

# imputation server url
url = 'https://imputationserver.sph.umich.edu/api/v2'
token = 'YOUR-API-TOKEN';

# add token to header (see authentication)
headers = {'X-Auth-Token' : token }

# get all jobs
r = requests.get(url + "/jobs", headers=headers)
if r.status_code != 200:
    raise Exception('GET /jobs/ {}'.format(r.status_code))

# print all jobs
for job in r.json():
    print('{} [{}]'.format(job['id'], job['state']))
```

## Monitor Job Status

### /jobs/{id}/status

### Example: curl

Command:

```sh
TOKEN="YOUR-API-TOKEN";

curl -H "X-Auth-Token: $TOKEN" https://imputationserver.sph.umich.edu/api/v2/jobs/job-20160504-155023/status
```

Response:

```json
{
  "application":"Michigan Imputation Server (Minimac4) 1.5.8",
  "applicationId":"minimac4",
  "deletedOn":-1,
  "endTime":1462369824173,
  "executionTime":0,
  "id":"job-20160504-155023",
  "logs":"",
  "name":"job-20160504-155023",
  "outputParams":[],
  "positionInQueue":0,
  "running":false,
  "startTime":1462369824173,
  "state":5
  ,"steps":[]
}
```

## Monitor Job Details

### /jobs/{id}

### Example: curl

```sh
TOKEN="YOUR-API-TOKEN";

curl -H "X-Auth-Token: $TOKEN" https://imputationserver.sph.umich.edu/api/v2/jobs/job-20160504-155023/
```
