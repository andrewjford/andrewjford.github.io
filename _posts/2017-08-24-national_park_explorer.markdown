---
layout: post
title:  "National Park Explorer"
date:   2017-08-24 20:39:43 +0000
---

This is a brief overview of my React final project which can be found at [https://natl-park-explorer.herokuapp.com/](https://natl-park-explorer.herokuapp.com/).

National Park Explorer is a web application that allows users to explore U.S. National Parks through a map based interface. It uses React for the front end, a Rails API back end, Leaflet and Mapbox to help provide this experience.

<img src="https://i.imgbox.com/uIIboyKL.png" width="100%">

I used the open source Javascript library Leaflet to create a full screen map for the main page. The tiling for the map comes from Mapbox. I used the [react-leaflet](https://github.com/PaulLeCam/react-leaflet) library to assist in integrating the Leaflet map library into React.

Leaflet allows use of markers and interactivity that users expect from web based maps. One challenge with using Leaflet with React was allowing Leaflet components to access React routes. Look out for a future blog post on how I worked around that issue.

The data for the map is pulled from the U.S. National Park Service (NPS) Data [API](https://www.nps.gov/subjects/digital/nps-data-api.htm). Rather than making calls to the NPS API for every action, I stored the most used information on a database that integrates with the Rails back end. The React front end makes all its requests to the Rails back end API. The back end either provides the necessary data from the local database, or it makes a request to the NPS API.

I used Redux to manage the state for React in my app. See [this](https://andrewjford.github.io/2017/08/17/redux_reducers/) post to read about a bug I encountered related to Redux.

Users can rate parks out of 5 stars. Zooming in on a park populates the map with that park's visitors centers (if provided by the NPS API). A detailed page of the park including photos, description and links can be accessed by clicking the details link.

<img src="https://i.imgbox.com/rz7NPhoL.png" width="100%">

For future implementations I would like to include user registration to allow users to make notes on and track parks. Also I would like a better image display system on the park detail page. Currently image links are pulled from the NPS API but there are issues with the size and availability of those images.
