# SoundCloud case analysis

### 1) Elements description

- Services:

1. API Gateway - secure access to the data and further services.
2. Load balancer - balance load of incoming requests across EC2 instances.
3. Compute for request handling - several instances of EC2 to route and handle incoming requests.
4. DB - relational DB to store user info.
5. Music storage - S3 blob storage for music. To upload music POST request is being sent. To download - GET request is being used.
6. Metadata storage - DynamoDB service, to store information about songs metadata, like: sond_id, owner_id, genres, path to S3 storage, etc.
7. Storage for likes and track progress - DynamoDB service, to store likes or music listening progress.
8. Redshift - data warehouse for analytical purposes. Here is being stored detailised info about likes, listening progress. We can retrieve aggregated data over long time period.

- Motivation for REST API Monolith architecture:

1. Simplicity of implementation
2. Result consistency

### 2) Functional requirements satisfaction

1. User can upload to platform audiofiles: POST method from an App => API Gateway => Load balancer => EC2 => Store audio-file on S3 => Store audio-file metadata on DynamoDB
2. User can listen audiofiles (personal or uploaded by other users): GET method from an App => API Gateway => Load balancer => EC2 => GET audio-file metadata on DynamoDB by song_id => Get audio-file on S3 by path for an audio-file path from metadata
3. User can like tracks: POST method from an App => API Gateway => Load balancer => EC2 => Store likes into Dynamo DB.
4. User can see listening stats of personally uploaded audiofiles: GET method from an App => API Gateway => Load balancer => EC2 => Get aggregated data from Redshift.
