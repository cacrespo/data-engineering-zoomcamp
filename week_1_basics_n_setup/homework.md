## Week 1 Homework

In this homework we'll prepare the environment 
and practice with terraform and SQL


## Question 1. Google Cloud SDK

Install Google Cloud SDK. What's the version you have? 

To get the version, run `gcloud --version`

```console
carlos@3520:~$ gcloud --version
Google Cloud SDK 370.0.0
```

## Google Cloud account 

Create an account in Google Cloud and create a project.

DONE 😄


## Question 2. Terraform 

Now install terraform and go to the terraform directory (`week_1_basics_n_setup/1_terraform_gcp/terraform`)

After that, run

* `terraform init`
* `terraform plan`
* `terraform apply` 

Apply the plan and copy the output (after running `apply`) to the form.

```console
carlos@3520:~$ terraform apply
var.project
  Your GCP Project ID

  Enter a value: 572846541943


Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  # google_bigquery_dataset.dataset will be created
  + resource "google_bigquery_dataset" "dataset" {
      + creation_time              = (known after apply)
      + dataset_id                 = "trips_data_all"
      + delete_contents_on_destroy = false
      + etag                       = (known after apply)
      + id                         = (known after apply)
      + last_modified_time         = (known after apply)
      + location                   = "europe-west6"
      + project                    = "572846541943"
      + self_link                  = (known after apply)

      + access {
          + domain         = (known after apply)
          + group_by_email = (known after apply)
          + role           = (known after apply)
          + special_group  = (known after apply)
          + user_by_email  = (known after apply)

          + view {
              + dataset_id = (known after apply)
              + project_id = (known after apply)
              + table_id   = (known after apply)
            }
        }
    }

  # google_storage_bucket.data-lake-bucket will be created
  + resource "google_storage_bucket" "data-lake-bucket" {
      + force_destroy               = true
      + id                          = (known after apply)
      + location                    = "EUROPE-WEST6"
      + name                        = "dtc_data_lake_572846541943"
      + project                     = (known after apply)
      + self_link                   = (known after apply)
      + storage_class               = "STANDARD"
      + uniform_bucket_level_access = true
      + url                         = (known after apply)

      + lifecycle_rule {
          + action {
              + type = "Delete"
            }

          + condition {
              + age                   = 30
              + matches_storage_class = []
              + with_state            = (known after apply)
            }
        }

      + versioning {
          + enabled = true
        }
    }

Plan: 2 to add, 0 to change, 0 to destroy.

Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value: yes

google_bigquery_dataset.dataset: Creating...
google_storage_bucket.data-lake-bucket: Creating...
google_storage_bucket.data-lake-bucket: Creation complete after 3s [id=dtc_data_lake_572846541943]
google_bigquery_dataset.dataset: Creation complete after 4s [id=projects/572846541943/datasets/trips_data_all]

Apply complete! Resources: 2 added, 0 changed, 0 destroyed.
```


It should be the entire output - from the moment you typed `terraform init` to the very end.

## Prepare Postgres 

Run Postgres and load data as shown in the videos

We'll use the yellow taxi trips from January 2021:

```bash
wget https://s3.amazonaws.com/nyc-tlc/trip+data/yellow_tripdata_2021-01.csv
```

You will also need the dataset with zones:

```bash 
wget https://s3.amazonaws.com/nyc-tlc/misc/taxi+_zone_lookup.csv
```

Download this data and put it to Postgres

DONE 😄

## Question 3. Count records 

How many taxi trips were there on January 15?

Consider only trips that started on January 15.

** 53024 trips **

```sql 
SELECT COUNT(*)
FROM PUBLIC.YELLOW_TAXI_TRIPS
WHERE DATE(TPEP_PICKUP_DATETIME) = '2021-01-15'
```

## Question 4. Largest tip for each day

Find the largest tip for each day. 
On which day it was the largest tip in January?

Use the pick up time for your calculations.

(note: it's not a typo, it's "tip", not "trip")

** It was at 2021-01-20: 1140.44 😮 **

```sql 
WITH JANUARY AS
	(SELECT DATE(TPEP_PICKUP_DATETIME) AS DATE,
			MAX(TIP_AMOUNT) AS LARGEST_TIP
		FROM PUBLIC.YELLOW_TAXI_TRIPS
		WHERE DATE(TPEP_PICKUP_DATETIME) BETWEEN '2021-01-01' AND '2021-01-31'
		GROUP BY 1
		ORDER BY 2 DESC)
SELECT *
FROM JANUARY
LIMIT 1;
```

## Question 5. Most popular destination

What was the most popular destination for passengers picked up 
in central park on January 14?

Use the pick up time for your calculations.

Enter the zone name (not id). If the zone name is unknown (missing), write "Unknown" 

** The most popular destination from CP on January 14 was Upper East Side South **

```sql 
SELECT B."Zone" AS ZONE_UP,
	C."Zone" AS ZONE_DEST,
	COUNT(*) AS N_TRIPS
FROM PUBLIC.YELLOW_TAXI_TRIPS A
LEFT JOIN PUBLIC.ZONES B ON A."PULocationID" = B."LocationID"
LEFT JOIN PUBLIC.ZONES C ON A."DOLocationID" = C."LocationID"
WHERE DATE(TPEP_PICKUP_DATETIME) = '2021-01-14'
	AND B."Zone" = 'Central Park'
GROUP BY 1,2
ORDER BY 3 DESC
LIMIT 3 -- check for equal count
```

## Question 6. Most expensive locations

What's the pickup-dropoff pair with the largest 
average price for a ride (calculated based on `total_amount`)?

Enter two zone names separated by a slash

For example:

"Jamaica Bay / Clinton East"

If any of the zone names are unknown (missing), write "Unknown". For example, "Unknown / Clinton East". 

** Answer: 'Alphabet City / Unknown' with an average of 2292.4. **

```sql 
SELECT
	COALESCE(B."Zone",'Unknown') || ' / ' || COALESCE(C."Zone", 'Unknown') AS pickup_dropoff_pair,
	AVG(TOTAL_AMOUNT) AS AVERAGE_AMOUNT
FROM PUBLIC.YELLOW_TAXI_TRIPS A
LEFT JOIN PUBLIC.ZONES B ON A."PULocationID" = B."LocationID"
LEFT JOIN PUBLIC.ZONES C ON A."DOLocationID" = C."LocationID"
GROUP BY
	B."Zone",
	C."Zone"
ORDER BY 2 DESC
```

## Submitting the solutions

* Form for submitting: https://forms.gle/yGQrkgRdVbiFs8Vd7
* You can submit your homework multiple times. In this case, only the last submission will be used. 

Deadline: 26 January (Wednesday), 22:00 CET


## Solution

Here is the solution to questions 3-6: [video](https://www.youtube.com/watch?v=HxHqH2ARfxM&list=PL3MmuxUbc_hJed7dXYoJw8DoCuVHhGEQb)

