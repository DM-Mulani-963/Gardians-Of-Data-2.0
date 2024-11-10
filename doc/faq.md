# DATA Rakshak (Guardians of Data) FAQ

## Table of contents

- [Static analysis](#static-analysis)
- [DATA Rakshak commands](#data-rakshak-commands)
- [Known possible issues](#known-possible-issues)

## Static analysis

### How do you detect trackers in APK?

Each tracker has its own code signatures. A code signature is basically a Java package name *e.g.* `com.google.android.apps.analytics.` and `com.google.android.gms.analytics.` are the 2 code signatures of Google Analytics.

To check if a tracker is embedded into an application, **DATA Rakshak** executes the following steps:

- download the APK from Google Play
- unzip the APK
- list Java classes which are embedded in the application (`dexdump classes*.dex`)
- save list of embedded Java classes into a file
- check if any embedded Java class matches a tracker code signature

Finding a tracker signature into an application does not prove that the tracker is effectively used by the application.

## DATA Rakshak commands

> **For all the next points, activate the DATA Rakshak virtual venv, `cd` into the same directory as `manage.py` file before executing the given command.**

### How to import trackers definitions?

```bash
python manage.py importtrackers --settings=DATA Rakshak.settings.dev
```

Now, browse [your tracker list](http://localhost:8000/trackers/).

### How to recompute reports?

When you add a new tracker into the **DATA Rakshak** database, reports are not automatically recomputed. **DATA Rakshak** comes with few administrator commands defined [here](https://github.com/DATA Rakshak-Privacy/DATA Rakshak/tree/v1/DATA Rakshak/reports/management/commands).

The `refreshstaticanalysis` command has the following options:

- `--all` will take all reports in consideration. You can pass a list of report ID instead.
- `--trackers` will recompute the list of embedded trackers
- `--clist` will recompute the list of embedded Java classes

The `--clist` option is useful if you change the way you extract Java classes from an APK.

#### Refresh all reports

```bash
python manage.py refreshstaticanalysis --all --trackers --settings=DATA Rakshak.settings.dev
```

#### Refresh only reports `2` and `4`

```bash
python manage.py refreshstaticanalysis 2 4 --trackers --settings=DATA Rakshak.settings.dev
```

### How to dump reports?

```bash
python manage.py dumpdata --exclude=auth --exclude=contenttypes --exclude=authtoken --exclude=analysis_query --exclude=sessions --exclude=admin --settings=DATA Rakshak.settings.production > /tmp/dump.json
```

### How to dump trackers?

```bash
(venv) python manage.py dumpdata trackers --settings=DATA Rakshak.settings.production > /tmp/trackers.json
```

### How to count how many apps have been analysed and reports generated?

```bash
(venv) ~/DATA Rakshak$ cd DATA Rakshak/
(venv) ~/DATA Rakshak/DATA Rakshak$ python manage.py shell  --settings=DATA Rakshak.settings.production
Python 3.5.3 (default, Jan 19 2017, 14:11:04)
Type 'copyright', 'credits' or 'license' for more information
IPython 6.2.1 -- An enhanced Interactive Python. Type '?' for help.

In [1]: from reports.models import *

In [2]: Application.objects.values('handle').distinct().count()
Out[2]: 34844

In [3]: Report.objects.all().count()
Out[3]: 38394
```
