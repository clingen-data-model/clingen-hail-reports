
# Instructions for setting up Hail to read gnomAD

## Install hailctl

Hail works best through anaconda python. Use the following link to install Hail into an anaconda environment.

https://hail.is/docs/0.2/getting_started.html#installing-hail-on-mac-os-x-or-gnu-linux-with-pip

## Service account permissions

By default, Hail will use the default project service account. By default, this service account does not have the necessary permissions to manage Dataproc resources. So either add those permissions to the default service account, or create a new service account for Dataproc/Hail. For example `hail-service-account`.

The permissions required are `Dataproc Worker`. (I also added `Dataproc Administrator` and `Dataproc Service Agent`, neither of which were sufficient).

For interaction with files from notebook, it needs `Service Usage Consumer` (contains `serviceusage.services.use`)

## Create cluster

If a non-default service account is being used to work with Hail (for example a hail/dataproc-specific one created above), that can be specified by appending `--service-account=<NAME>` to the argument list, before the final positional parameter for the cluster name. Here `<NAME>` is the name of the service account, for example `hail-service-account@<PROJECT>.iam.gserviceaccount.com`

Set the project name to the appropriate Google Cloud project name.

If a zone is set in `gcloud` (`gcloud config [get|set-value] compute/zone`), the hail commands will automatically use that. Otherwise, a zone can be specified on all commands with `--zone`.

The cluster idle timeout and number of workers can also be configured. It is also important to set the region (and zone) to be in the same location as the data being read, if possible. Otherwise network egress fees will be incurred when reading any files over the network.

The command to start a medium sized cluster (1 master, 2 primary workers, 2 preemptible workers) is below. Using this cluster configuration, the report file is generated in approximately 15-16 minutes.

```
$ PROJECT=clingen-dev
$ hailctl dataproc start --project $PROJECT --region us-east1 --zone us-east1-b --max-idle 1h --num-preemptible-workers 2 --service-account=hail-service-account@$PROJECT.iam.gserviceaccount.com cg-hail
```

Note that the `--max-idle` flag may not interpret jupyter notebook interaction as interactions with the cluster if no dataproc jobs are being scheduled, and may treat work as being idle, and shut down the cluster earlier than expected.

Example, for a project called `clingen-dev`:

```
$ hailctl dataproc start --project clingen-dev --region us-east1 --zone us-east1-b --max-idle 1h --num-preemptible-workers 2 --service-account=hail-service-account@clingen-dev.iam.gserviceaccount.com cg-hail
```

Small cluster (note: not efficient for running a large scale analysis):

```
$ hailctl dataproc start --project clingen-dev --region us-east1 --zone us-east1-b --max-idle 4h --num-preemptible-workers 0 --service-account=hail-service-account@clingen-dev.iam.gserviceaccount.com --master-machine-type=n1-highmem-2 --worker-machine-type=n1-standard-2 cg-hail
```

The gnomAD v3.1 dataset constitutes a significant increase in data size over the gnomAD v2.x series. This data must be downloaded onto dataproc worker nodes. Because the dataset is so large, network bandwidth itself can be a bottleneck in run time. For this reason, it may be beneficial to use a larger number of smaller instances rather than a smaller number of larger instances, as spreading data partitions across more instances means less bottlenecking on the network links of each instance.

```
hailctl dataproc start --project clingen-dev --region us-east1 --zone us-east1-b --max-idle 4h --num-workers 4 --num-preemptible-workers 4 --service-account=hail-service-account@clingen-dev.iam.gserviceaccount.com --master-machine-type=n1-highmem-2 --worker-machine-type=n1-standard-2 cg-hail
```


### Requester Pays

Some buckets (ex: gnomad) may be set as 'requester-pay', which requires a user project to be set in storage API calls. Hail can automatically set this by setting an option on cluster creation:

```
--requester-pays-allow-all (allow any requester pay API calls)
--requester-pays-allow-buckets (comma separated buckets, avoid unintended charges from downloads)
```

```
$ hailctl dataproc start --project clingen-dev --region us-east1 --zone us-east1-b --max-idle 4h --num-preemptible-workers 0 --service-account=hail-service-account@clingen-dev.iam.gserviceaccount.com --master-machine-type=n1-highmem-2 --worker-machine-type=n1-standard-2 --requester-pays-allow-buckets gnomad-public-requester-pays cg-hail`
```

## Connect to cluster

For interactive use, start a jupyter notebook on the dataproc cluster. This will set up a tunnel to the notebook server on a local machine port. The notebook in this repository can then be uploaded via the Jupyter web interface.

```
$ hailctl dataproc connect cg-hail notebook
```


# NOTES

Mid-size cluster with updated instance types and more CPU on workers. May not be ideal. Much of time spent appears to be downloading gnomad partitions into worker processes. Should experiment with doing a copying of gnomad table into HDFS first and loading the table into Hail from there. Copy might be too slow though. HDFS size dictated by combined size of primary worker boot disks (or local ssds).
```
hailctl dataproc start --project clingen-dev --region us-east1 --zone us-east1-b --max-age 8h --service-account=hail-service-account@clingen-dev.iam.gserviceaccount.com --num-preemptible-workers 4 --num-workers 2 --worker-machine-type e2-highcpu-16 --master-machine-type e2-highmem-8 cg-hail
```
