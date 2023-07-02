# SoundCloud case analysis

### 1) Functional requirements

1. Listen music/songs online
2. Make playlists
3. Find music per artist or song name
4. Download music on local device

### 2) Non-functional requirements

1. Consistency => Music stream should be consistent, without glitches and losses
2. Portability => Service could be used as in web, as well as on PC/mobile app
3. Reusability => Uploaded or downloaded songs, list of favorites should be read for next use session
4. Security => Private user info should be protected from leakage

### 3) Load and resources evaluation

1. Total active users (per month) - 175 milion
2. Daily active users - 5.8 milion
3. Possible concurrent requests - 100 per second
4. Available tracks on SoundCloud - 150 milion
5. Average song weight in 320 KB/s - 15 MB
6. Expected song storage - 150000000 x 15 MB = 2145 TB
7. Expected user info per day = 50 B
8. Storage for user info - (50 x 30 x 175000000) = 230 GB per month
