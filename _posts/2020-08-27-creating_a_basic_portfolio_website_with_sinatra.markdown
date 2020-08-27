---
layout: post
title:      "Creating a basic portfolio website with Sinatra"
date:       2020-08-27 23:17:26 +0000
permalink:  creating_a_basic_portfolio_website_with_sinatra
---


Since Flatiron has a focus on comphrensive projects at the end of each module, I felt it was relevant to base my Sinatra project around how to display these projects we have been working on. In creating a Sinatra web app, my aim was to allow users to show case the basic elements of their projects.

An initial challenge was deciding how much information to allow - beyond a title and description, what else would provide details while looking streamlined? How many programming language options were too many?

Ultimately I settled on a few details and 11 common programming languages (with more to come). Mapping out the project details for showing, editing, and creating new projects then became straightforward. 

The next challenge became about verifiying that a user has logged in, that routes are protected, and users can't access projects created by others - I feel that coming into this unit, I had a basic understanding of http routes, but the 'redirect' commands and display of .erb files really cemented my understanding. My main design decision was to include edit and delete on the project page itself, rather than in the index list of projects, so that users can view and edit details more closely. 

Rounding out the project, I added a display message for any unknown routes, and added a simple css layout for convenient viewing of projects and project details. I'm looking forward to building on this knowlege of Sinatra with the next unit for Ruby on Rails.
