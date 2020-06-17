# gke-log-volume
A bash script to measure log output volume on a Google Kubernetes Engine (GKE) node. Useful for estimating total log volume when designing log aggregation systems, for measuring the log output of specific deployments, or for detecting log volume
imbalance across nodes.

## Prerequisites

Requires Google Cloud command line client (gcloud) installed and authorized to the target cluster. The kubectl GKE client is useful for enumerating nodes in the cluster but not required to run this script. Note that the script executes the `wc` and `du` commands on the remote logfiles using sudo, so the user must have sudo permission on the nodes. This is usually the case
if you are connecting with a project owner/editor/viewer role.

## Install

Just clone the repo or simply copy the `gke-log-volume` script somewhere on your path and mark it executable (`chmod 755 gke-log-volume`). See below for usage and examples.

## Use

```
NAME
  gke-log-volume - connects to a GKE (Google Kubernetes Engine) node and
  measures lines/bytes written to log files over a configurable interval.

SYNOPSIS
  gke-log-volume [OPTIONS] NODE

DESCRIPTION
  gke-log-volume enumerates the log files in /var/log/containers, and takes
  a snapshot of the current line count and size in bytes for each. It then
  sleeps for a configurable interval and takes another measurement, before
  reporting the results in a format that is easy to consume and analyze.

POSITIONAL ARGUMENTS
  NODE
    Name of the GKE node to connect to.

OPTIONAL ARGUMENTS
  -h|--help
    Display this help. All other arguments are ignored.

  -d|--duration SECONDS
    Sets the length of the period between measurements in seconds. Defaults
    to 30 seconds.

  -p|--project PROJECT
    The name of the Google Cloud project in which the node is located.
    Defaults to the current value returned by 'gcloud config get-value
    core/project'.

  -z|--zone ZONE
    The name of the Google Cloud zone in which the node is located.
    Defaults to the current value returned by 'gcloud config get-value
    compute/zone'.

  -e|--exclude REGEXP
    Sets a regexp used to exclude logs. Log filenames that match this
    regexp will not be included in the output or totals.

  -l|--include REGEXP
    Sets a regexp used to include logs. Log filenames that match this
    regexp will be included, all others will be ignored.

  -u|--user USER
    Overrides the user that is used to authenticate the connection.

  -i|--internal-ip
    If set the ssh connection will be made to the node's internal IP.

  -a|--active-only
    If set logs that have 0 lines written during the test will be ignored.

  -n|--node-only
    If set individual log results will be omitted and only node totals
    will be output.

  -q|--quiet
    If set disable all output except the results.

OUTPUT
  The script outputs results for each log and for the node as a whole
  (except in the case where --node-only is set) in the format:

  [name] [lines written] [bytes written] [lines/second] [bytes/second]
```
## Limitations

The script does not attempt to detect or deal with log rotation. If you get negative numbers back from a node this is due to the container log rotating and the /var/log/containers symlink getting updated to point to the new file. Log volume can vary widely over time depending on the type of workload. A number of other system level considerations can affect the rate and timing of writes to the log files. For all of these reasons multiple test runs should be performed and negative values discarded. In general this tool is meant to give an approximation of total volume and relative volume across nodes. It should not be relied on for exact numbers.

## PR's welcome!
