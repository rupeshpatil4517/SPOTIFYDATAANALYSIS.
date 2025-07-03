# SPOTIFYDATAANALYSIS.
SQL-based project for exploring Spotify music data
create database sp;
use sp;
select * from spotifycsv;
-- Exploratory Data Analysis
select count(*) from spotifycsv;
---------------------------------------------------------
select count(distinct artist) from spotifycsv;
---------------------------------------------------------
select count( distinct album_type ) from spotifycsv;
---------------------------------------------------------
select duration_min from spotifycsv;
---------------------------------------------------------
select max(duration_min) from spotifycsv;
---------------------------------------------------------
select min(duration_min) from spotifycsv;
---------------------------------------------------------
select* from spotifycsv 
where duration_min = 0;
---------------------------------------------------------
select distinct channel from spotifycsv;
---------------------------------------------------------
select distinct most_playedon from spotifycsv;
UPDATE spotifycsv
SET likes = 7220875
WHERE album = 'Demon Days';
---------------------------------------------------------
-- Add a new column named Genre (type VARCHAR(50)) to the spotifycsvtable.
ALTER table spotifycsv 
add GENRE VARCHAR(50);

-----------------------------------------------------------------------------------------
-- Q.1 Retrieve the names of all tracks that have more than 1 billion streams.
select* from spotifycsv
where stream > 1000000000;
-----------------------------------------------------------------------------------------
-- Q.2 list all albums along with their respective artists.
select album,artist
from spotifycsv;
select distinct album,artist from spotifycsv
order by 1;
-----------------------------------------------------------------------------------------
-- Q.3 Get the total number of comments for tracks where licensed = TRUE
select SUM(comments) AS total_comments
 from spotifycsv
where licensed = 'TRUE';
-----------------------------------------------------------------------------------------
-- Q.4 Find all tracks that belongs to the album type single.
select* from spotifycsv
where album_type = 'single';
-----------------------------------------------------------------------------------------
-- Q.5 Count the total number of tracks by each artist.
select artist,count(*) AS total_no_songs
from spotifycsv
group by artist
order by 2 asc;
----------------------------------------------------------------------------------------
-- Q.6 Calculate the average danceability of tracks in each album.
select album,
avg(danceability)
from spotifycsv
group by 1
order by 2 desc;
----------------------------------------------------------------------------------------
-- Q.7 Find the top 5 tracks with the highest energy values.
select track,MAX(energy)
 from spotifycsv
group by 1
order by 2 desc
LIMIT 5;
----------------------------------------------------------------------------------------
-- Q.8 List all tracks along with their views and likes where official_video = TRUE.
select track,
sum(views),
sum(likes)
from spotifycsv
WHERE official_video = 'TRUE'
group by 1;
----------------------------------------------------------------------------------------
-- Q.9 For each album, calculate the total views of all associated tracks.
select album,
track,
sum(views)
from spotifycsv
group by 1,2
order by 3 desc;
----------------------------------------------------------------------------------------
-- Q.10 Retrive the track names that have been streamed on Spotify more than Youtube.
select track,
most_playedon
from spotifycsv
where most_playedon = "spotify"
group by 1;

----------------------------------------------------------------------------------------
-- Q.11 Find the top 3 most- viewed tracks for each artist using window functions.
SELECT *
FROM (
    SELECT 
        artist,
        track,
        SUM(views) AS total_views,
        DENSE_RANK() OVER (PARTITION BY artist ORDER BY SUM(views) DESC) AS view_rank
    FROM spotifycsv
    GROUP BY artist, track
) AS ranked_tracks
WHERE view_rank <= 3;

----------------------------------------------------------------------------------------
-- Q,12 write a query to find tracks where the liveness score is above average.
select avg(liveness) -- 0.19
from spotifycsv;
select track,liveness
 from spotifycsv
WHERE liveness > 0.19;
-----------------------------------------------------------------------------------------
-- Q.13 Use a WITH clause to calculate the difference between the highest and lowest energy values for tracks in each album.
with cte
AS
(select album,
max(energy) AS highest_energy,
min(energy) AS lowest_energy
from spotifycsv
group by 1
)
select album,
highest_energy - lowest_energy AS energy_diff
from cte
order by 2 desc;
-----------------------------------------------------------------------------------------
-- Q.14 find tracks where the energy-to-liveness ratio is greater than 1.2.
SELECT Track, artist,Energy, Liveness,
       Energy / Liveness AS energy_liveness_ratio
FROM spotifycsv
WHERE Energy / Liveness > 1.2;
-----------------------------------------------------------------------------------------
-- Q. 15 calculate the cumulative sum of likes for each artist using window function
SELECT 
    Artist,
    Track,
    Likes,
    SUM(Likes) OVER (PARTITION BY Artist ORDER BY Track desc) AS cumulative_likes
FROM spotifycsv;
-----------------------------------------------------------------------------------------
-- Q. 16 Show the total Spotify streams per artist.
SELECT Artist, 
SUM(Stream) AS total_streams
FROM spotifycsv
GROUP BY Artist
ORDER BY total_streams DESC;
-----------------------------------------------------------------------------------------
-- Q.17 Show top 5 most-liked official videos.
select track,likes
from spotifycsv
where official_video = 'true'
order by 2 desc
limit 5;
-----------------------------------------------------------------------------------------
-- Q. 18 List tracks that appear more than once.
SELECT Track
FROM spotifycsv
GROUP BY Track
HAVING COUNT(Track) > 1;
------------------------------------------------------------------------------------------
-- Q.19 Create a stored procedure called GetArtistTracks that takes an artist name as input and returns the list of track names and album names by that artist from the spotifycsv table.
DELIMITER //

CREATE PROCEDURE GetArtistTracks(IN artist_name VARCHAR(100))
BEGIN
    SELECT track , album
    FROM spotifycsv
    WHERE artist = artist_name;
END //

DELIMITER ;
CALL GetArtistTracks('coldplay');
-------------------------------------------------------------------------------------------------------
create table music(
id int primary key,
singer varchar(20),
album varchar(50)
);
INSERT INTO music VALUES
(1,"Gorillaz","Demon Days"),
(2,"coldplay","Parachutes"),
(3,"Metallica","lover");
select* from music;
-- Q.20 write a sql query to album and singer name.
SELECT id,singer
FROM music
JOIN spotifycsv
ON spotifycsv.album = music.album;
-------------------------------------------------------------------------------------------------------
-- Q.21 Find tracks that have licensed = TRUE AND duration_min is greater than 3 minutes.
SELECT track,
Licensed,
Duration_min
 FROM spotifycsv
WHERE licensed = 'TRUE' AND duration_min > 3;
-----------------------------------------------------------------------------------------------------------
-- Q.22 Create a trigger that logs track names and timestamps whenever a new track is inserted.
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

-- Check the log
SELECT * FROM track_log;
----------------------------------------------------------------------------------------------------------------------




