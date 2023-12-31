# Glovo case analysis

### 1) Resource load analysis

- Used websites for evaluation:
  - <https://expandedramblings.com/index.php/glovo/>
  - <https://about.glovoapp.com/press/glovo-delivered-2021-report-highlights-global-delivery-and-consumer-trends/>
  - <https://d3.harvard.edu/platform-digit/submission/meet-glovo-the-app-that-will-deliver-anything-to-your-door/#:~:text=Founded%20in%20Barcelona%20in%202015%2C%20Glovo%20is%20an,to%20you%20%E2%80%9Cwithin%20minutes%E2%80%9D%20%28average%20of%2028%20minutes%29>.

- Terminology:
  - Instead of using restauraunt/shop owners I can use vendors.

- Shops and restauraunts data.
  - Shops and restauraunts amount for 2021: 130,000.
  - Each shop has approximately 100 positions (including shops, restaurants, etc.). It means that we need to store info about 13,000,000 positions.
  - Lets assume that each position has picture and textual description. Average compressed picture size is 5 MB. 
  - Each position has in average 5 photos. Approximately, description in Glovo has 200 symbols, which means 200 B.
  - Total shops and restauraunts data amount: (13000000 x 25000000) + (13000000 x 200) = 325 TB

- User data (including regular users, verndors and couriers).
  - Total amount of users: 20,000,000.
  - Average user info weight: 3 KB.
  - Total user data amount: 20000000 x 3000 = 60 GB.

- Data generated by couriers and orders:
  - Non-functional requirement: refresh frequency of courier`s geo-location should be no more than 5 seconds.
  - Number of daily orders: 1,000,000. Lets assume that each order is being rated.
  - Each rating and review data is about 100 B. It will be included in orders details and in dedicated ratings storage.
  - Average data amount per order, including order details and review: 1 KB.
  - Average delivery duration: 28 minutes (1,560 seconds). Average time-window amount to be send from courier: 312.
  - Average data amount generated by courier each time-window: 100 B.
  - Total data generated by couriers per month : 100 x 312 x 1000000 x 30 = 936 GB.
  - Total data generated by orders per month: 1000 x 1000000 x 30 = 30 GB
  - Total data generated by rating of each order per month: 100 x 1000000 x 30 = 3 GB

- Conclusions about storage after resource evaluation:
  - Storage for user info: AWS RDS, as not so big data amount.
  - Storage for orders operational storage: AWS DynamoDB, cause of fast response and high workload is required.
  - Storage for historical orders: AWS Redshift, for long-term storage and big batches of data analysis.
  - Storage for shops and restauraunts info: AWS DynamoDB for vendors metadata and AWS S3 for pictures.
  - Storage for data generated by couriers: AWS S3, windows from stream in Parquet files.
  - Storage for ratings of couriers and vendors: Redis and DynamoDB with orders data for long-term, as we require fast response and amount of rating data will be not big.

### 2) Elements description

- Services:
  - Service for user info processing
  - Service for courier tracking
  - Service for rating processing
  - Service for order processing
  - 3rd party service for payment processing
  - Service for shops and restaurants info storage
  - API Gateway as security service

- Service for user info processing: responsible for app users info processing, including regular users, couriers, shop/restauraunt owners.
  - AWS Lambda - function to write data to AWS RDS where user info data is being stored. Can be triggered using HTTP request from an app.
  - AWS RDS - RDS where user info is being stored.

- Service for courier tracking: responsible for receiving incoming data stream from courier and sending courier location to regular user and vendor.
  - AWS SQS - SQS service to process incoming data stream from courier and send it to AWS Kinesis Data Stream.
  - AWS Kinesis Data Stream - stream service that accepts incoming data stream from SQS.
  - AWS Kinesis Firehose - stream consumer and ETL tool of data that comes from Kinesis Data Stream.
  - AWS S3 - long-term storage of courier activity that is being fixed in windows by Kinesis Firehose.
  - AWS SNS - notification service that consumes incoming data stream from Kinesis Firehose and send courier location to regular user and vendor.

- Service for rating processing: responsible for ratings processing.
  - AWS SQS - SQS service to process incoming requests of rating update or requests to retrieve rating.
  - AWS Lambda - function to update ratings both cached in Redis and long-term stored in DynamoDB.
  - Redis - memory cache to store ratings data and to retrieve solely this info.

- Service for order processing: responsible for incoming requests for order and sending order statuses updates to subscribed users.
  - AWS SQS - SQS service to process incoming order requests and send (un)subscribe request for notifications about order statuses.
  - AWS Lambda for incoming order processing - function that writes incoming request to DynamoDB. Also, this function process request for payment. When payment received: invo is being sent to DynamoDB and order record will be updated.
  - AWS DynamoDB - NoSQL storage for orders data. Record of order can be updated when status is changed - for instance, when order was paid, delivered or responsible persons for order, as vendor and courier, were rated.
  - Managed Airflow instance - service for ETL that on schedule grabs data from DynamoDB and stores it in Redshift.
  - AWS Redshift - DWH in cloud, to store order details for long-term period and for analytics purposes.
  - AWS Lambda that is triggered on new orders - function that is being triggered when new order comes to DynamoDB or status of already existing orders was changed.
  - AWS SNS - notification service that consumes incoming data from Lambda and sends notifications to subscribed users. User can (un)subscribe via SQS in this service.

- 3rd party service for payment processing: some service that processes requests for order payments and returns payment status to our service that process orders.

- Service for shops and restaurants info storage: responsible for storage and fetch of vendors data.
  - Load balancer: balances load cause of incoming requests for static content - vendors info. Frequently requested data is being cached to minimize load on servers.
  - EC 2: Compute for vendor info storage, helps to store or fetch vendors data.
  - AWS DynamoDB: metadata storage, where details about product are being stored: vendor_id, product_id, price, description, path to picture in S3, etc.
  - S3: storage for products pictures.

- API Gateway as security service: central point through which incoming requests are coming to our system.

1. Kinesis Data Stream - streaming service that consumes incoming data from the app and produce it to speed and long-terms storages.
2. Kinesis Firehose - ETL of incoming streaming data. This service will help to apply simple transformation of incoming data and store it in the final storage.
3. S3 - blob storage for long-term storage of data. Data will be stored in batches (windows).
4. Timestream - time-series database service that suits for consumption of incoming streaming data and further analytical purposes.
5. Quicksight - BI service that consumes data in Timestream storage and visualize it for the final user.

- Motivation for micro-service architecture:

1. High reliability
2. High speed
3. No risk of overload
4. Distribution of responsibility for tasks

### 3) Functional requirements satisfaction

1. Vendor can create shop/restauraunt profile, populate products list, products description and prices for them:
   1. Vendor via App makes request to service for vendor invo processing
   2. API Gateway
   3. Load Balancer
   4. EC2 as metadata processing service
   5. Store metadata in DynamoDB
   6. Store products picture in S3
2. Courier can receive notifications in real-time about available orders. Orders should be near couriers geolocation. Courier can accept orders for completion:
   1. Courier via App makes request to service for order processing
   2. API Gateway
   3. AWS SQS receives request for orders with specified geo-location
   4. AWS SNS receives request for notifications subscribtion for specific geo-location
   5. In case of new orders - notification to courier will be sent as Lambda function being triggered on new orders.
3. User can search products or dishes from available restauraunts or shops. User can make order and pay for it:
   1. User via App makes request to service for vendor invo processing
   2. API Gateway
   3. Load balancer: if vendor info is being cached - return it to the user. If not cached - we are moving forward
   4. EC2: process user request for particular shor or restauraunt and its products
   5. DynamoDB and S3: retrieve data about vendor and its products from the storages
   6. User via App makes request to service for order processing, in order to make an order
   7. AWS SQS: receives incoming request about new order and makes further request to Lambda.
   8. Lambda for incoming order processing: sends request for payment to third-party service and writes initial order record to DynamoDB.
   9. DynamoDB: stores order record. When the order was paid - Lambda function updates order status into DynamoDB.
   10. Lambda that is triggered on new orders or status updates: is triggered when new record is being written. Sends notification to the user that order was successfuly paid.
4. User can see in real time courier and the order movement on the map:
   1. User via App makes request to service for courier tracking
   2. AWS SQS: receives incoming request for courier tracking. In request is specified courier_id and order_id for tracking purposes. SQS send subscribe request to SNS.
   3. AWS SNS: subscribe on courier activity. When new data comes - updates are sent to the subscribed user.
5. Vendor can track couriers activity that are delivering orders from this shop/restauraunt:
   1. Vendor via App makes request to service for courier tracking
   2. AWS SQS: receives incoming request for courier tracking. In request is specified courier_id and vendor_id for tracking purposes. SQS send subscribe request to SNS.
   3. AWS SNS: subscribe on courier activity. When new data comes - updates are sent to the subscribed vendor.
6. Each restauraunt, shop and courier has own rating, which is being formed by users. Rating should be refreshed in real-time:
   1. User receives notification on App via AWS SNS from order processing service that order is completed, so user can set rating for vendor and courier.
   2. User via App makes request to service for rating processing.
   3. AWS SQS: receives incoming request for rating update, sends data with rating and review text to lambda function.
   4. AWS Lambda: updates rating data both in Redis cache and in DynamoDB from order processing service (in case of Redis unavailability).
   5. Redis: stores ratings and reviews of vendors and couriers. In case if user needs to retrieve ratings - we can use Redis for this purposes.
