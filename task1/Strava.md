# Strava case analysis

### 1) Functional requirements

1. Track physical activity info of the user (route, elevation, speed, heart rate, etc.)
2. Share completed activities with other users
3. Do social activity in the app, like (un)follow users, post photos or videos, give kudos
4. Buy premium subscription

### 2) Non-functional requirements

1. Consistency => Gathered info about physical activity should be precise enough
2. Portability => Service could be used as in web, as well as on mobile app or third-party services
3. Reliability => Physical activity history, uploaded photos or videos, should be stored for long time
4. Security => Private user info or sensitive state info should not be revealed

### 3) Load and resources evaluation

1. Total active users (per month) - 95 milion
2. Daily active users - 3.01 milion
3. Possible concurrent requests - 35 per second
4. Expected user info per day = 500 B
5. Storage for user info - (500 x 30 x 95000000) = 1.29 TB per month
