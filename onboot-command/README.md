# On-boot commands

This initialization action can be used as either an initialization action or as a startup-script,
with the latter enabling support for commands which need to be re-run on reboot. This
initialization action adds polling for the cluster itself to be fully healthy before issuing
a command, effectively allowing "post-initiailization" actions which can include further setup
or submitting a job.

## Using this initialization action

For now, the script relies on polling the Dataproc API to determine an authoritative state
of cluster health on startup, so requires the `--scopes cloud-platform` flag; do not use
this initialization action if you are unwilling to grant your Dataproc clusters' service
account with the `cloud-platform` scope.

## Examples

### Submit a single job with cluster and turn off cluster on completion

    ```bash
    export CLUSTER_NAME=${USER}-shortlived-cluster
    gcloud dataproc clusters create ${CLUSTER_NAME} \
        --scopes cloud-platform \
        --initialization-actions gs://dataproc-initialization-actions/onboot-command/dataproc-master-onboot.sh \
        --metadata onboot-command=" \
            gcloud dataproc jobs submit hadoop \
                --cluster ${CLUSTER_NAME} \
                --jar file:///usr/lib/hadoop-mapreduce/hadoop-mapreduce-examples.jar \
                teragen 10000 /tmp/teragen; \
            gcloud dataproc clusters delete -q ${CLUSTER_NAME}"
