# SoundCloud case analysis

### 1) Elements description

- Services:
1. Load balancer - requests router, to handle POST/PATCH/GET API requests.
2. DB - relational DB to store user info.
3. Music storage - S3 blob storage for music. To upload music POST request is being sent. To download - GET request is being used. 
4. Dynamo DB - key-value storage for user stats, such as likes or music listening progress. Can be used to retrieve statistics per user or song.
5. Redis - mem-cache DB for recent info storage of user stats. Stats are being cached from Redshift. If stats are not cached - request for fresh data is being send to Redshift. 
6. Redshift - data warehouse for analytical purposes. Here is being stored detailised info about likes, listening progress. Also, data from DB with user info is being fetched via ETL.

- Motivation for such architecture:
1. Simplicity of implementation
2. Result consistency 

### 2) Functional requirements satisfaction

1. User can upload to platform audiofiles: POST request to S3 storage with music.
2. User can listen audiofiles (personal or uploaded by other users): GET request to S3 storage with music.
3. User can like tracks: POST request to Dynamo DB.
4. User can see listening stats of personally uploaded audiofiles: GET request to Redis. If no cached stats in Redis - request is being redirected to Redshift.
