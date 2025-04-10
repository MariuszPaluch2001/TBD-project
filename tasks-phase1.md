IMPORTANT ❗ ❗ ❗ Please remember to destroy all the resources after each work session. You can recreate infrastructure by creating new PR and merging it to master.
  
![img.png](doc/figures/destroy.png)

1. Authors:

   310866

   [***link to forked repo***](https://github.com/MariuszPaluch2001/TBD-project)
   
2. Follow all steps in README.md.

3. In boostrap/variables.tf add your emails to variable "budget_channels".

4. From avaialble Github Actions select and run destroy on main branch.
   
5. Create new git branch and:
    1. Modify tasks-phase1.md file.
    
    2. Create PR from this branch to **YOUR** master and merge it to make new release. 
    
    ***place the screenshot from GA after succesfull application of release***
![img.png](doc/figures/task-phase1-release-destroy-screen.png)


6. Analyze terraform code. Play with terraform plan, terraform graph to investigate different modules.

    ```bash
    terraform graph -type=plan | dot -Tpng >graph.png
    ```

    ***describe one selected module and put the output of terraform graph for this module here***
   
7. Reach YARN UI
   
   ***place the command you used for setting up the tunnel, the port and the screenshot of YARN UI here***
   
8. Draw an architecture diagram (e.g. in draw.io) that includes:
    1. VPC topology with service assignment to subnets
    2. Description of the components of service accounts
    3. List of buckets for disposal
    4. Description of network communication (ports, why it is necessary to specify the host for the driver) of Apache Spark running from Vertex AI Workbech
  
    ***place your diagram here***

9. Create a new PR and add costs by entering the expected consumption into Infracost
For all the resources of type: `google_artifact_registry`, `google_storage_bucket`, `google_service_networking_connection`
create a sample usage profiles and add it to the Infracost task in CI/CD pipeline. Usage file [example](https://github.com/infracost/infracost/blob/master/infracost-usage-example.yml) 

   ***place the expected consumption you entered here***
  ```yaml
    version: 0.1
    resource_usage:
    google_artifact_registry_repository.registry:
        storage_gb: 50 # Total data stored in the repository in GB
    google_storage_bucket.tbd_data_bucket:
        storage_gb: 150                   # Total size of bucket in GB.
        monthly_class_a_operations: 4000 # Monthly number of class A operations (object adds, bucket/object list).
        monthly_class_b_operations: 2000 # Monthly number of class B operations (object gets, retrieve bucket/object metadata).
        monthly_egress_data_transfer_gb:  # Monthly data transfer from Cloud Storage to the following, in GB:
        same_continent: 40  # Same continent.
        worldwide: 40     # Worldwide excluding Asia, Australia.
        asia: 10           # Asia excluding China, but including Hong Kong.
        china: 10            # China excluding Hong Kong.
        australia: 10       # Australia.
    google_storage_bucket.tbd_code_bucket:
        storage_gb: 100
        monthly_class_a_operations: 1000
        monthly_class_b_operations: 500
        monthly_egress_data_transfer_gb: 100
    google_service_networking_connection.my_connection:
        monthly_egress_data_transfer_gb:  250
  ```

   ***place the screenshot from infracost output here***
![img.png](doc/figures/infracost-output.png)

10. Create a BigQuery dataset and an external table using SQL

    ```sql
    CREATE SCHEMA `tbd-2025l-310979.dataset`
      OPTIONS (
        location = 'europe-west1'
      )
    ```
    ![](doc/figures/dataset_created.png)

    ```sql
    CREATE EXTERNAL TABLE `tbd-2025l-310979.dataset.shakespeare`
      OPTIONS (
        format ="orc",
        uris = ['gs://tbd-2025l-310979-data/data/shakespeare/*.orc']
      );
    ```
    ![](doc/figures/external_table_created.png)
    ![](doc/figures/select_shakespeare.png)
   
    ### Why does ORC not require a table schema?

    ORC doesn’t need a table schema because it saves the schema inside the file itself. So when you read an ORC file, it already knows what the data looks like: column names, types, etc.

    ### Other

    To create the external table, it was necessary to fix the `spark-job.py` from the next task, as creating this table requires data along with the format (the ORC data format includes this).

11. Find and correct the error in spark-job.py

    The issue was an incorrect path to the bucket where the processed data was supposed to be saved. The correct bucket path was provided, and the problem was resolved.
    It could have been found in the line:  
    ```
    DATA_BUCKET = "gs://tbd-2025l-9900-data/data/shakespeare/"
    ```

12. Add support for preemptible/spot instances in a Dataproc cluster

    [main.tf](./modules/dataproc/main.tf)
    ```
    spot_worker_config {
      num_instances = var.spot_worker_count
      machine_type  = var.machine_type
      disk_config {
        boot_disk_type    = "pd-standard"
        boot_disk_size_gb = 100
      }

      preemptibility = "SPOT"
    }
    ```

    [variables.tf](./modules/dataproc/variables.tf)
    ```
    variable "spot_worker_count" {
      type        = number
      default     = 0
      description = "Spot worker nodes count"
    }
    ```

    A variable `spot_worker_count` was created with a default value of 0. If a value greater than 0 is set, we can launch spot instances of virtual machines.