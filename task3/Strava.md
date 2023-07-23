# Strava case analysis

### 1) Elements description

- Services:

1. Load balancer - balance work load of incoming requests.
2. DB - storage for user info.
3. Kinesis Data Stream - streaming service that consumes incoming data from the app and produce it to speed and long-terms storages.
4. Kinesis Firehose - ETL of incoming streaming data. This service will help to apply simple transformation of incoming data and store it in the final storage.
5. S3 - blob storage for long-term storage of working-out sessions data. Data will be stored in batches (windows).
6. Timestream - time-series database service that suits for consumption of incoming streaming data and further analytical purposes. Here will be stored working-out sessions data for analysis purposes.
7. Quicksight - BI service that consumes data in Timestream storage and visualize it for the final user.

- Motivation for Kappa-like architecture:

1. High reliability
2. High speed
3. No risk of overload

### 2) Functional requirements satisfaction

1. Running tracker: send data to Kinesis Data Stream and further store in S3 and Timestream.
2. Revisit previous run sessions (with visualization): use visualization in Quicksight to get session stats with visualization.
3. Retrieve aggregated stats about working-out sessions for selected time-period: get data from S3 for desired time-window(s).
