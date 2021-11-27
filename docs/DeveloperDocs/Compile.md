# Developer Guide

This page shows you how to build, test and install the project.

## Build

To build the project you need [Go](https://golang.org/) Version 1.16.x and [Java](https://java.com/) Version 11 on your development machine.
On your target machine, the samba server you want to monitor, you need [samba](https://www.samba.org/) and [systemd](https://www.freedesktop.org/wiki/Software/systemd/) installed.

To build the software change to the repositories directory and run:

```sh
./gradlew getBuildName build preparePack
```

In case you want to work on the man pages you need to install [ronn](https://github.com/rtomayko/ronn) on your development machine.
To create the man pages out of the `*.ronn` source files run:

```sh
build/CreateManPage.sh 
```

## Run Tests locally

To execute the unit tests you can run:

```sh
./gradlew test
```

To execute the integration tests you can run:

```sh
./test/integrationTest/scripts/RunIntegrationTests.sh
```

## Manual installation

For manual install on the `target` machine do the following copy:

```sh
scp ./tmp/samba-exporter_<version>/* root@<target>:/
```

Now login to your target machine and run the commands below to enable the services and create the needed user and group:

```sh
systemctl daemon-reload
systemctl enable samba_statusd.service
systemctl enable samba_exporter.service
addgroup --system samba-exporter
adduser --system --no-create-home --disabled-login samba-exporter
adduser samba-exporter samba-exporter
updatedb                                                          # In case you created and copied the man pages as well
```

Finally you are abel to start the services:

```sh
systemctl start samba_statusd.service
systemctl start samba_exporter.service
```

Test the `samba_exporter` by requesting metrics with `curl`:

```sh
curl http://127.0.0.1:9922/metrics 
```

The output of this test should look something like this:

```txt
# HELP go_gc_duration_seconds A summary of the pause duration of garbage collection cycles.
# TYPE go_gc_duration_seconds summary
go_gc_duration_seconds{quantile="0"} 0
...
# HELP promhttp_metric_handler_requests_in_flight Current number of scrapes being served.
# TYPE promhttp_metric_handler_requests_in_flight gauge
promhttp_metric_handler_requests_in_flight 1
# HELP promhttp_metric_handler_requests_total Total number of scrapes by HTTP status code.
# TYPE promhttp_metric_handler_requests_total counter
promhttp_metric_handler_requests_total{code="200"} 0
promhttp_metric_handler_requests_total{code="500"} 0
promhttp_metric_handler_requests_total{code="503"} 0
# HELP samba_client_count Number of clients using the samba server
# TYPE samba_client_count gauge
samba_client_count 0
# HELP samba_individual_user_count The number of users connected to this samba server
# TYPE samba_individual_user_count gauge
samba_individual_user_count 0
# HELP samba_locked_file_count Number of files locked by the samba server
# TYPE samba_locked_file_count gauge
samba_locked_file_count 0
# HELP samba_pid_count Number of processes running by the samba server
# TYPE samba_pid_count gauge
samba_pid_count 0
# HELP samba_satutsd_up 1 if the samba_statusd seems to be running
# TYPE samba_satutsd_up gauge
samba_satutsd_up 1
# HELP samba_server_up 1 if the samba server seems to be running
# TYPE samba_server_up gauge
samba_server_up 1
# HELP samba_share_count Number of shares used by clients of the samba server
# TYPE samba_share_count gauge
samba_share_count 0
```