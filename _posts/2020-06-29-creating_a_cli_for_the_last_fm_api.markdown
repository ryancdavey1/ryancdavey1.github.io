---
layout: post
title:      "Creating a CLI for the last.fm API"
date:       2020-06-29 06:42:26 +0000
permalink:  creating_a_cli_for_the_last_fm_api
---


I'm always trying to find new music. Which made the focus of the CLI project fairly straightforward - I wanted to create a command line interface that would allow the user to find new songs, playlists, and genres.

What wasn't easy was finding a music source that allowed for web scraping. I did start off with certain websites, but they contained a lot of dynamic content that is not compatible with scraping. After scraping the initial html, the content detials could not be properly accessed; song, playlist, and genres were visible inside one entire object, but couldn't be seperated easily into individual objects.

Time to switch it up.

I switched over to last.fm API, looking at the [chart of top tracks](http://www.last.fm/api/show/chart.getTopTracks) for a sample of current songs. The API provided sufficient informatino upon initial call, but I did have a challenge with accessing tracks within the initial json object. After working through the initial API call, creating individual song objects needed to be completed. 

Song details could then be collected through web scraping and the song's particular url on the last.fm website, and proved to be more straightfoward given the static content. Bringing all of the information together, I was now able to display the top 50 songs from the chart, and display a song's title, artist, length, etc.

One addition that I felt was necessary was creating a playlist while the interface is being used - songs can be looked up, but what about storing them? I finished up the project by adding options to select a song and add it to a current playlist for the session, hopefully simulating real-time playlist creation!

