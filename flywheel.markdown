
# Flywheel

Some projects use data stored on Flywheel, which can be programmatically accessed through the Flywheel SDK. First, follow the directions [here](https://docs.flywheel.io/hc/en-us/articles/360008162214-Installing-the-Command-Line-Interface-CLI-) to install the Flywheel CLI, then `fw login`. Then, install the python SDK with `pip install flywheek-sdk`.

Then setup a client:
```py
import flywheel
fw = flywheel.Client()
```
Then data can be downloaded for every subject:
```py
project = fw.lookup('cnet/7T-MS-agespan')
subjects = project.subjects()
for sub in subjects:
    if sub.label in targets:
        for ses in sub.sessions():
            full_ses = fw.get(ses.id)
            files.append(full_ses)
fw.download_tar(files, 'data.tar')
```
or you can create a query:
```py
acquisitions = fw.search({'structured_query': "acquisition.label CONTAINS something", 'return_type': 'acquisition'}, size=10000)
files = []
for acq in acquisitions:
    files.append(acq.acquisition)
fw.download_tar(files, 'data.tar')
```
Queries can be constructed through the Flywheel website (Search > Advanced Search).

Files can be uploaded back to Flywheel with the `upload_file_to_acquisition` method:
```py
acquisitions = fw.search({'structured_query': "acquisition.label CONTAINS something", 'return_type': 'acquisition'}, size=10000)
fname = "t1.nii.gz"
for acq in acquisitions:
    acq_files = fw.get(acq.acquisition.id)
    file_names = [f['name'] for f in acq_files.to_dict()['files']]
    to_upload = os.path.join('output', acq.subject.code, acq.session.label, acq.acquisition.label, fname)
    if not os.path.exists(to_upload):
        print("Missing", to_upload)
        continue
    if fname in file_names:
        print('Already uploaded', to_upload)
        continue
    try:
        fw.upload_file_to_acquisition(acq.acquisition.id, to_upload)
    except:
        print("Failed to upload", to_upload)

```