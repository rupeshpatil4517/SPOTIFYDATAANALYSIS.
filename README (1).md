
# 🎧 Spotify Data Analysis using SQL

This project showcases comprehensive SQL-based data analysis on Spotify's music dataset. It includes data cleaning, exploratory data analysis, aggregation, window functions, stored procedures, joins, and triggers to generate actionable insights from music streaming data.

---

## 🛠 Tools & Technologies

- **Database**: MySQL
- **Interface**: MySQL Workbench
- **Language**: SQL
- **Dataset Format**: CSV (imported as `spotifycsv`)

---

## 📊 SQL Project Code

```sql
CREATE DATABASE sp;
USE sp;
SELECT * FROM spotifycsv;

-- Exploratory Data Analysis
SELECT COUNT(*) FROM spotifycsv;
SELECT COUNT(DISTINCT artist) FROM spotifycsv;
SELECT COUNT(DISTINCT album_type) FROM spotifycsv;
SELECT MAX(duration_min), MIN(duration_min) FROM spotifycsv;
SELECT * FROM spotifycsv WHERE duration_min = 0;
SELECT DISTINCT channel FROM spotifycsv;
SELECT DISTINCT most_playedon FROM spotifycsv;

-- Data Cleaning
UPDATE spotifycsv SET likes = 7220875 WHERE album = 'Demon Days';
ALTER TABLE spotifycsv ADD GENRE VARCHAR(50);

-- Analysis Queries
SELECT * FROM spotifycsv WHERE stream > 1000000000;
SELECT DISTINCT album, artist FROM spotifycsv ORDER BY 1;
SELECT SUM(comments) AS total_comments FROM spotifycsv WHERE licensed = 'TRUE';
SELECT * FROM spotifycsv WHERE album_type = 'single';
SELECT artist, COUNT(*) AS total_no_songs FROM spotifycsv GROUP BY artist ORDER BY 2 ASC;
SELECT album, AVG(danceability) FROM spotifycsv GROUP BY 1 ORDER BY 2 DESC;
SELECT track, MAX(energy) FROM spotifycsv GROUP BY 1 ORDER BY 2 DESC LIMIT 5;
SELECT track, SUM(views), SUM(likes) FROM spotifycsv WHERE official_video = 'TRUE' GROUP BY 1;
SELECT album, track, SUM(views) FROM spotifycsv GROUP BY 1,2 ORDER BY 3 DESC;
SELECT track FROM spotifycsv WHERE most_playedon = "spotify" GROUP BY 1;

-- Window Functions
SELECT * FROM (
  SELECT artist, track, SUM(views) AS total_views,
         DENSE_RANK() OVER (PARTITION BY artist ORDER BY SUM(views) DESC) AS view_rank
  FROM spotifycsv GROUP BY artist, track
) AS ranked_tracks
WHERE view_rank <= 3;

-- Conditional Analysis
SELECT track, liveness FROM spotifycsv WHERE liveness > (SELECT AVG(liveness) FROM spotifycsv);
WITH cte AS (
  SELECT album, MAX(energy) AS highest_energy, MIN(energy) AS lowest_energy
  FROM spotifycsv GROUP BY 1
)
SELECT album, highest_energy - lowest_energy AS energy_diff FROM cte ORDER BY 2 DESC;

SELECT track, artist, energy, liveness, energy / liveness AS energy_liveness_ratio
FROM spotifycsv WHERE energy / liveness > 1.2;

-- Cumulative Sum
SELECT artist, track, likes,
       SUM(likes) OVER (PARTITION BY artist ORDER BY track DESC) AS cumulative_likes
FROM spotifycsv;

-- Aggregation
SELECT artist, SUM(stream) AS total_streams FROM spotifycsv GROUP BY artist ORDER BY total_streams DESC;
SELECT track, likes FROM spotifycsv WHERE official_video = 'TRUE' ORDER BY likes DESC LIMIT 5;
SELECT track FROM spotifycsv GROUP BY track HAVING COUNT(track) > 1;

-- Stored Procedure
DELIMITER //
CREATE PROCEDURE GetArtistTracks(IN artist_name VARCHAR(100))
BEGIN
    SELECT track, album FROM spotifycsv WHERE artist = artist_name;
END //
DELIMITER ;
CALL GetArtistTracks('coldplay');

-- Joins Example
CREATE TABLE music (
    id INT PRIMARY KEY,
    singer VARCHAR(20),
    album VARCHAR(50)
);
INSERT INTO music VALUES
(1, "Gorillaz", "Demon Days"),
(2, "coldplay", "Parachutes"),
(3, "Metallica", "lover");

SELECT id, singer
FROM music
JOIN spotifycsv ON spotifycsv.album = music.album;

-- Filtered Query
SELECT track, licensed, duration_min
FROM spotifycsv
WHERE licensed = 'TRUE' AND duration_min > 3;

-- Trigger
CREATE TABLE track_log (
    log_id INT AUTO_INCREMENT PRIMARY KEY,
    track_name VARCHAR(255),
    inserted_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
DELIMITER //
CREATE TRIGGER log_new_track
AFTER INSERT ON spotifycsv
FOR EACH ROW
BEGIN
    INSERT INTO track_log (track_name)
    VALUES (NEW.track);
END //
DELIMITER ;
INSERT INTO spotifycsv (track, album, artist, stream, duration_min, licensed)
VALUES ('Echoes of Time', 'Timeless Vibes', 'Luna Sound', 850000, 3.8, 'TRUE');

SELECT * FROM track_log;
```

---

## 📬 Contact

**Rupesh Patil**  
📧 rp45171909@gmail.com  
🔗 [LinkedIn](https://www.linkedin.com/in/rupesh-patil-62bb0731a)  
💻 [GitHub](https://github.com/rupeshpatil4517)
