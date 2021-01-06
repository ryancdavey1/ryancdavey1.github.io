---
layout: post
title:      "Understanding the complexity of rails "
date:       2020-11-03 01:29:45 -0500
permalink:  understanding_the_complexity_of_rails
---


The Ruby on Rails project was by far the most complex project I have worked on from scratch. This felt like the first project in which I had to spend a majority of the time thinking about what shouldn't be there, rather than a majority of the time spent thinking about what to add to the code.

My project idea started off fairly simple - users would be able to log into the rails application and create new artwork (for the purposes of this project, only text details were saved) with an index page of all artwork or just the user's artwork visible (every artwork has Starry Night by Vincent Van Gogh as its image).

This proved to be a smaller step in terms of effort to create, and I distinguished between artworks listed on the site and a user's own artwork with this relationship in the `User` model:
`has_many :produced_artworks, foreign_key: :artist_id, class_name: 'Artwork'`
This single line of code made it much easier to list a user's creations, and know that the databas was tracking it through the artist_id foreign key. 

The challenging part came with adding features to the artwork: I figured it would be interesting to allow user's to create bids on listed artwork (even one's own artwork). But I tried to make things too complex by attaching an artwork's id directly to a bid, after the link had been selected to create a new bid - this link would be available on the artwork page, or by simply typing the route `/artworks/:id/bids/new` in the browser.

Ultimately, I decided to reduce the complexity by adding a `collection_select` form to the new bids page, which allowed the user to select between all artworks listed on the site - this felt like a better user experience in case a user changed their mind on which artwork they wanted to create a bid for. I felt that I spent a long time coming to the conclusion that the new_bid_path was the best approach, rather than adding the artwork id in an artwork_new_bid_path.

Finally came the most complext part of the project - protecting routes. The one route I left public was the artwork index page, so that anyone could see descriptions of art listed on the site. With the Sinatra project, I had solved the protecting routes part by adding logic to each controller action checking for the current user and a logged in status.

There was a lot of trial and error this time to create helper functions for checking logged in status, as well as redirecting an unauthenticated use. I finally settled on using `skip_before_action` sparingly with a few of the routes, which meant that I could rely on the ApplicationController utilizing my `redirect_if_not_logged_in` method to check for a user's authentication - way easier than adding logic to each controller, and once again, making me think of what shouldn't be there in each controller.
