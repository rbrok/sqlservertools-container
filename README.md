
## About
[mssql-scripter](https://github.com/microsoft/mssql-scripter) wasn't working for me anymore after upgrading to Fedora 34.

Since my database setup and test logic absolutely needs mssql-scripter, I started to search and found other users with the [same problem](https://github.com/microsoft/mssql-scripter/issues/236).\
mssql-scripter requires a very old libssl1.0 library, which is not available at newer distributions like Fedora 34 (Fedora 33 still provides a compatibility version [compat-openssl10](https://koji.fedoraproject.org/koji/buildinfo?buildID=1574056)).\
The workaround is to provide mssql-scripter and its dependencies in a container, but the solutions in the problem link above didn't satisfy me. My concern is the very old libssl version in Debian Jessie.

After trying different combinations of base containers and installed packages, my current best solution is Debian Stretch Slim which provides a libssl1.0.2 package.\
If I find time, I'll try it again with a minimal Fedora 33 container and the above compatibility version (I should have done that first...).

The container also provides [sqlcmd, bcp](https://docs.microsoft.com/en-us/sql/linux/sql-server-linux-setup-tools?view=sql-server-ver15) and [mssql-cli](https://github.com/dbcli/mssql-cli/).

## Installation
I have tested the container with [Podman](http://docs.podman.io/en/latest/) 3.1.2 under Fedora 34. Docker and Podman are sharing the same command line interface and you can replace 'podman' with 'docker' in the following commands, if Podman is not available for you.

Build the container with:
```shell
podman build -t sqlservertools -f debian-stretch-containerfile
```

To test the container in an interactive mode:
```shell
podman run --rm -it sqlservertools
```

The following command executes mssql-scripter inside the container:
```shell
podman run --rm -it sqlservertools mssql-scripter
```

But mssql-scripter expects connection informations:
```shell
podman run --rm -it sqlservertools mssql-scripter -S servername -U username -d databasename
```
This repository provides short bash scripts to run eg. mssql-scripter.\
You could copy these scripts into one your path folders

 ```shell
chmod -v u+x ./scripts/*
cp -v ./scripts/* $HOME/.local/bin
```

 and than you can run mssql-scripter something like that:
```shell
mssql-scripter -S servername -U username -d databasename
```

The script delegates MSSQL_SCRIPTER host environment variables into the container. If you execute eg.:
```shell
export MSSQL_SCRIPTER_PASSWORD=MyPassword
```
mssql-scripter will use that password.

## TODOs
* bcp isn't tested yet
* I couldn't etablish a connection with mssql-scripter via a connection string
* create a script to copy the bash scripts into the path
