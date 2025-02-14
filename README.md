# RapidMiner Python package

This Python package allows you to interact with RapidMiner Studio and Server. You can collaborate using the RapidMiner repository and leverage the scalable Server infrastructure to run processes. This document shows examples on how to use the package. Additional notebook files provide more advanced examples. There is an API document for each classes: [Studio](docs/api/Studio.md), [Server](docs/api/Server.md), [Scoring](docs/api/Scoring.md).

## Table of contents

- [Requirements](#requirements)
- [Known current limitations](#known-current-limitations)
- [Overview](#requirements)
- [Installation](#installation)
- [Studio](#studio)
- [Server](#server)
- [Scoring](#scoring)

## Requirements

* RapidMiner Studio *9.3.0* for Studio class
* RapidMiner Server *9.3.0* for Server class
* Python Scripting extension *9.3.0* installed for both Studio and Server, download it from the [Marketplace](https://marketplace.rapidminer.com/UpdateServer/faces/product_details.xhtml?productId=rmx_python_scripting)

## Known current limitations

* Python version: 
  * Extensive tests were only carried out using *Python 3.7*, but earlier versions are expected to work as well.
  * Python 2 is currently not supported.
* Server read and write methods can only handle data currently, Studio read and write methods can handle other objects as well.
* Studio and Server processes guarantee reproducibility. That means you should always get the same result after a version update. The same feature *cannot be guaranteed* when using this Python library (the library depends on other libraries that our not in our control).
* Server with [SAML authentication](https://redirects.rapidminer.com/web/saml-authentication) is not yet supported.

## Overview

Both Studio and Server classes provide a read and a write method for reading / writing data and other objects, and a run method to run processes. The method signatures are the same, with somewhat different extra parameters.

Studio class requires a local Studio installation and is suitable in the following cases:
* Implementing certain data science steps in Python using your favorite IDE or notebook implementation. You may even use the resulting code afterwards in a RapidMiner process within an *Execute Python* operator.
* You are using coding primarily, but you want to incorporate methods that are impemented in a RapidMiner process.
* Creating batch tasks that also interact with the repository and / or run processes.

Server class connects directly to a Server instance without the need of a Studio installation. It is suitable in the following cases:
* Collaborating with RapidMiner users, sharing data easily.
* Calling, running, scheduling processes on the RapidMiner Server platform from a local script.

## Installation

The library can be installed from this repository:

- install in one step:

        $ pip install git+https://github.com/rapidminer/python-rapidminer.git

- or clone and install:

        $ git clone https://github.com/rapidminer/python-rapidminer.git
        $ cd python-rapidminer
        $ python setup.py install

## Studio

You need to have a locally installed RapidMiner Studio instance to use this class. The only thing you need to provide is your installation path. Once that is specified, you can read from and write data or other objects to any configured repository. You can also run processes from files or from the repository. In this section, we show you some examples on how to read and write repository data and run processes. For more advanced scenarios see the included [IPython notebook](examples/studio_examples.ipynb) and the [documentation of the `Studio` class](docs/api/Studio.md).

Note that each `Studio` method starts a Studio instance in the background and stops it when it is done. It is not recommended to run multiple instances in parallel, e.g. on different Notebook tabs. If you have several RapidMiner extensions installed, all of them will be loaded each time, that may lead to longer runtime. Provide multiple parameters to a read or write call, if possible, to avoid the startup overhead. 

First you need a `Connector` object to interact with Studio. Once you have that, you can read and write data or run a process with a single line. To create a `Studio` `Connector` object, run the following code:

```python
import rapidminer
connector = rapidminer.Studio("/path/to/you/studio/installation")
```

where you replace `"/path/to/you/studio/installation"` with the location of your Studio installation. In case of Windows, a typical path is `"C:/Program Files/RapidMiner/RapidMiner Studio"` - note that you should either use forward "/" as separators or put an `r` before the first quote character to indicate raw string
. In case of Mac, the path is usually `"/Applications/RapidMiner Studio.app/Contents/Resources/RapidMiner-Studio"`. Alternatively you can define this location via the `RAPIDMINER_HOME` environment variable.

##### Reading ExampleSet

Once you have a connector instance, you can read a RapidMiner ExampleSet in Python by running the following line:

```python
df = connector.read_resource("//Samples/data/Iris")
```

The resulting `df` is a `pandas` `DataFrame` object, which you can use in the conventional way.

##### Writing ExampleSet

You can save any `pandas` `DataFrame` object to a RapidMiner repository (or file) with the following command:

```python
connector.write_resource(df, "//Local Repository/data/mydata")
```

where `df` is the `DataFrame` object you want to write to the repository, and `"//Local Repository/data/mydata"` is the location where you want to store it.

##### Running a process

To run a process execute the following line:

```python
df = connector.run_process("//Samples/processes/02_Preprocessing/01_Normalization")
```

You will get the results as `pandas` `DataFrames`. You can also define inputs, and many more. For more examples, see the [examples notebook](examples/studio_examples.ipynb)

## Server

With `Server` class, you can directly connect to a local or remote Server instance without the need for any local RapidMiner (Studio) installation. You can read data from and write data to the Server repository and you can execute processes using the scalable Job Agent architecture. In this section, we show you some examples on how to read and write repository data and run processes. For more advanced scenarios see the included [IPython notebook](examples/server_examples.ipynb) and the [documentation of the `Server` class](docs/api/Server.md).

### Installation of Server API

The `Server` class requires a web service backend to be installed on RapidMiner Server. This is done automatically on the first instantiation of the Server class. However, in most cases, you need to create the repository folder for it manually. *Note that if you are the sole user of the Server instance or this Server API now and in the future, you can skip the steps below. Create a `Server` class from Python with the proper url. When you are asked for the repository path of the backend, just choose `/home/myrmuser/` as the target path.*

Please follow the steps below to install Server API backend. These steps are needed to be done only once. You can use RapidMiner Studio or the RapidMiner Server web UI. See screenshots for performing the steps below [from Studio](docs/install/ServerInstallFromStudio.md) or [from Server web UI](docs/install/ServerInstallFromServerUI.md).
1. Create a Server Repository folder (e.g. `repository_api` in the root folder).
1. Give **read**, **write**, **execute** permissions to the **users** group on this folder.
1. Specify this folder when you instantiate `Server` class from Python for the first time and it asks for the repository path.

The very first `Server` class instantiation can also be fully automated (thus, need for user input avoided), if you specify this repository path in the `processpath` parameter besides setting `url`, `username` and `password`.

On the RapidMiner Server web UI you can see the installed web service (*Processes*->*Web Services*). It has the name *Repository Service* by default, but you can change that with the optional parameter of `Server` class named `webservice`. If the web service is deleted, the next `Server` instantiation will re-create it.

### Usage of Server API

To create a `Server` `Connector` object, run the following code:

```python
import rapidminer
connector = rapidminer.Server("https://myserver.mycompany.com:8080", username="myrmuser")
```

where you replace `"https://myserver.mycompany.com:8080"` with the url of your Server instance and `"myrmuser"` with your username.

##### Reading ExampleSet

Once you have a connector instance, you can read a RapidMiner ExampleSet in Python by running the following line:

```python
df = connector.read_resource("/home/myrmuser/data/mydata")
```

The resulting `df` is a `pandas` `DataFrame` object, which you can use in the conventional way.

##### Writing ExampleSet

You can save any `pandas` `DataFrame` object to the Server repository with the following command:

```python
connector.write_resource(df, "/home/myrmuser/data/myresult")
```

where `df` is the `DataFrame` object you want to write to the repository, and `"/home/myrmuser/data/myresult"` is the location where you want to store it.

##### Running a process

To run a process execute the following line:

```python
df = connector.run_process("/home/myrmsuer/process/transform_data", inputs=df)
```

You will get the results as `pandas` `DataFrames`. You can also define multiple inputs, and other parameters. For more examples, see the [examples notebook](examples/server_examples.ipynb)

## Scoring

This class allows you to easily use a deployed [Real-Time Scoring](https://docs.rapidminer.com/server/scoring-agent/) service. You only need to provide the Server url and the particular scoring service endpoint to create a class instance. After that, you can use the predict method to do scoring on a Pandas DataFrame and get the result in a Pandas DataFrame as well. For instructions on how to deploy Real-Time Scoring on Server, please refer to its documentation.

```python
sc = rapidminer.Scoring("http://myserver.mycompany.com:8090", "score-sales/score1")
prediction = sc.predict(df)
```

where the scoring endpoint is at `"score-sales/score1"` that can be applied to the dataset `df`, and the resulting `prediction` is a `pandas` `DataFrame` object. You can find the `Scoring` class [documentation here](docs/api/Scoring.md).


