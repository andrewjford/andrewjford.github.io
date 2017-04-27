---
layout: post
title:  "NFL Team Schedule Scraper"
date:   2017-04-27 05:39:43 +0000
---


For my CLI project I chose to create a web scraper for NFL team schedules. Its main function is to display all 32 NFL teams and let the user drill down to each team to see the team's current schedule. It works off of the cbssports.com web pages.

Building the application went pretty smoothly for the initial level. I was able to quickly build a rough program that scraped team names and team website addresses, and entered them into team objects. I ran into a snag however when I got to setting up the second level of scraping. My scraper was set up to loop through each team object, get its team url (still on cbssports.com), and scrape for data on the 2017 schedule. The scraper did not return any data on the css selectors I had set up for the team webpages.

After much troubleshooting I eventually realized that the web addresses for each team that I had been storing from the initial scraping were faulty. They were concatenations of the original address 'cbssports.com/nfl/teams' and the team path '/nfl/teams/page/someteam'. Unfortunately this resulted in the '/nfl/teams' path being duplicated in the full web url, which caused the html to be pulled from just the original address. I quickly got that fixed and am now looking to finish up the interface of the application. I will definitely be more aware of address pathing in the future.
