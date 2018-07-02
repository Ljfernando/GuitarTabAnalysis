# Progressions
<i>Progressions</i> is a web-app the allows a user to find similar songs based on their chord progressions. You can either click on a given song or type in your own chord progression to find other similar songs. 

## Dataset
The data comes from https://www.ultimate-guitar.com/ where we scraped the top 1000 most popular tabs of all time. 

The scraped data is stored into a MySQL database ('UltimateGuitarTabs') with the following tables:
1. Tab_Data (id, song, artist, is_acoustic, tab_url)
    - id: Unique tab ID 
    - song: Name of song
    - artist: Name of artist
    - is_acoustic: Song is played acoustically (1), otherwise (0)
    - tab_url: URL to tab
2. Artists (name, url)
    - name: Name of artist
    - url: URL to artist profile
3. Hits (id, num_hits, votes, rating)
    - id: Unique tab ID
    - num_hits: # of times tab visited
    - votes: # number of votes received
    - rating: Avg rating out of 5
4. Chords (song, artist, tonality, capo, chords)
    - song: Name of song
    - artist: Name of artist
    - tonality: Original of song
    - capo: Fret to place capo
    - chords: String of full chord progression
Scraped on June 28th, 2018

## Chord Progression Analysis
Since music is temporal by nature, we treat each song as a stochastic process. In particular, we use markov chains to create transition matrices for each song, where chord names represent a state. For simplicity, we stuck with a 1st-order markov chain, though music itself requires higher or variable order models to truly highlight patterns. 

When a user clicks on a song, we calculate the euclidean distance of the respective transition matrix to all other songs within our dataset. Songs with smaller outcomes are ranked in ascending order and displayed on the web-app. The process is the same for when a user types in a chord progression.

In this project we focus more on the chord structure of a song as opposed to the chord names or key. For example, The Beatles: Let It Be follows the iconic I-V-vi-IV chord structure in the key of C. The chord names are C-G-Am-F. The more contemporary Someone Like You by Adele ranks #3 in similarity and has a chorus with chord progression A-E-F#m-D. This is also the I-V-vi-IV structure but now in the key of A. Therefore, we can highlight songs that have similar structure regardless of the original key. 

This would be a lot easier if everything was in the key of C. Unfortunately, (and rather fortunately for our ears) it is not. Thus we have to transpose each chord progression in our dataset to center around the key of C. Before doing this we must determine the key of each individual song first. 

UltimateGuitarTabs asks users to input the key of the song they are tabbing but it is not completely reliable. Some songs require a capo and the key and the indicated capo fret do not line up. Some users input the completely wrong key, Some users just don't enter the key at all. 

Luckily, basic music theory can help us out a with this. In the /data directory is a csv file titled key_table.csv. The first column contains a key. Then each row contains the possible chords that can be played if a song were in that particular key. All we need to do is count up the amount of times each chord in this table was played, then calculate the sum of each row. The row with the largest sum will be labeled as the key to that song. 

NOTE: This approach is a heuristic and by no means will give you the correct key every time but it's the best we can do for now.

Once we have the key for each song, we simply shift all the chords by n-half-steps depending on how many n-half-steps away the key is from C. This will 'standardize' all of our songs and simplify our distance metric calculation.

## Installation 
The UltimateGuitarTabs must first be stored in a mysql database. To do so, ensure you have MySQL installed on your machine. Then you can dump the Progressions/data/UltimateGuitarTabs.sql file into mysql with the following command:
'''
mysqldump -u root -p UltimateGuitarTabs < UltimateGuitarTabs.sql
'''

Or you can start mysql in your terminal and execute the command:
'''
source [full path to UltimateGuitarTabs.sql]
'''

Next enter the correct directory in the project:
'''
cd Progressions/Flask_app
'''

Install python packages
'''
pip install -r req.txt
'''

Run flask app
'''
python daemon.py
'''

Something like this should show up:

* Debug mode: off

* Running on http://0.0.0.0:5000/ (Press CTRL+C to quit)

Finally you can open up the index.html file in your browser or start up your own server to run the app. I run my code on Sublime and usually start up a Sublime Server to run the app.


