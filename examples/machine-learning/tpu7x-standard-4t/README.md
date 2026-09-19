# TPU 7x Blueprints

## TPU 7x (`tpu7x-standard-4t`) Slurm Cluster Deployment

This directory contains a Cluster Toolkit blueprint ([`tpu7x-slurm-blueprint.yaml`](tpu7x-slurm-blueprint.yaml)) and deployment configuration ([`tpu7x-slurm-deployment.yaml`](tpu7x-slurm-deployment.yaml)) for provisioning a Slurm 26.05 cluster with GCE-native TPU 7x (`tpu7x-standard-4t`) compute nodes and Google Cloud Managed Lustre storage mounted at `/home`.

Selective deployment and teardown for this blueprint (`image-env`, `image`, and `primary` groups) are documented centrally. See [examples/machine-learning/README.md](../README.md) for full details.

### Build the Cluster Toolkit `gcluster` binary

Follow the instructions [here](https://cloud.google.com/cluster-toolkit/docs/setup/configure-environment) to set up your Cluster Toolkit environment, including enabling required APIs and IAM permissions.

### Configure the deployment file

Edit [`tpu7x-slurm-deployment.yaml`](tpu7x-slurm-deployment.yaml) with your Terraform state bucket name, project, region, zone, reservation, and TPU topology settings:

```yaml
---
terraform_backend_defaults:
  type: gcs
  configuration:
    bucket: <TF_STATE_BUCKET_NAME>

vars:
  deployment_name: tpu7x-slurm
  project_id: <PROJECT_ID>
  region: <REGION>
  zone: <ZONE>
  tpu_cluster_size: 16
  tpu_accelerator_topology: 4x4x4
  tpu_reservation_name: <RESERVATION_NAME>
```

> **Note:**
>
> - If the GCS bucket specified in `terraform_backend_defaults.configuration.bucket` does not exist yet, `./gcluster deploy` (or `./gcluster create`) will create it automatically in your `project_id` and `region` (prompting for confirmation unless `--auto-approve` is passed).
> - Each `tpu7x-standard-4t` VM provides 4 TPU chips (`tpus_per_node = 4`), so the total chips in `tpu_accelerator_topology` divided by 4 must match `tpu_cluster_size` (for example, `2x2x2` = 8 chips = 2 VMs; `2x4x4` = 32 chips = 8 VMs; `4x4x4` = 64 chips = 16 VMs).

### Additional ways to provision

Cluster Toolkit also supports DWS Flex-Start and Spot VMs in addition to reservations:

- [For more information on DWS Flex-Start in Slurm](https://github.com/GoogleCloudPlatform/cluster-toolkit/blob/main/docs/slurm-dws-flex.md)
- [For more information on Spot VMs](https://cloud.google.com/compute/docs/instances/spot)

To use one of these alternative models, modify the `vars` section in `tpu7x-slurm-deployment.yaml` and replace `tpu_reservation_name` with one of the following:

- `tpu_enable_spot_vm: true` (for Spot VMs)
- `tpu_dws_flex_enabled: true` (for DWS Flex-Start)

### Deploy the Slurm Cluster

```bash
./gcluster deploy \
  -d examples/machine-learning/tpu7x-standard-4t/tpu7x-slurm-deployment.yaml \
  examples/machine-learning/tpu7x-standard-4t/tpu7x-slurm-blueprint.yaml \
  --auto-approve
```

If you have already built the custom Slurm image (`slurm-tpu-v7x-ubuntu2404`) in your project, you can deploy only the `primary` cluster group by skipping `image-env` and `image`:

```bash
./gcluster deploy \
  -d examples/machine-learning/tpu7x-standard-4t/tpu7x-slurm-deployment.yaml \
  examples/machine-learning/tpu7x-standard-4t/tpu7x-slurm-blueprint.yaml \
  --only primary \
  --auto-approve
```

## Clean Up

To destroy all resources created by the blueprint, run:

```bash
./gcluster destroy <DEPLOYMENT_NAME>
```

Replace `<DEPLOYMENT_NAME>` with the `deployment_name` specified in your deployment file.

**Note:** GCS buckets created for Terraform state are not deleted by the `./gcluster destroy` command and must be deleted manually.
