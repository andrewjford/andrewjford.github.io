---
layout: post
title:  "React/Redux State and Leaflet JS"
date:   2017-09-14 17:55:03 +0000
---


Leaflet JS is a map library I used in my [National Park Explorer](http://natl-park-explorer.netlify.com/) application. I used the `react-leaflet` library to integrate Leaflet with React. What I want to discuss today is managing state of a Leaflet map in a React project.

To set up a Leaflet map in React you first import the Map and TileLayer components from the react-leaflet library. The map component takes two important props, center and zoom, to set the initial position of the map. Here my map center is `37.0902, -95.7129` and zoom is set to 4.

<img src="https://images.imgbox.com/21/f3/AQJzJGsb_o.png" width="100%" alt="Map at zoom level 4" />


The TileLayer component must also be set in the Map to specify what map tiling to use; in this case I used an outdoors tiling from mapbox.

```
import { Map, TileLayer } from 'react-leaflet';

render() {
  return <Map center={this.props.map.center}
			zoom={this.props.map.zoom}
			<TileLayer
				// attribution= 'attributions to mapbox here'
				url="https://api.tiles.mapbox.com/v4/mapbox.outdoors/{z}/{x}/{y}.png?access_token={SOME_TOKEN}"
			/>
			</Map>
}
```

I passed the center and zoom values from props into the Map component in the above example. These only represent the initial position of the map, as Leaflet Map manages its own state for the position and zoom. The problem I ran into is how could I save the position (and zoom level) of the map when I changed pages away from the Map. When I changed pages, the Map component would get removed from the DOM and Leaflet would forget its last position. Going back to the map would re-render at the default initial position we passed above.

```
// this way causes issues
componentDidMount() {
  const leafletMap = this.leafletMap.leafletElement;

  leafletMap.on('moveend', () => {
    this.props.saveMapPosition(leafletMap.getCenter(), leafletMap.getZoom());
  });
}
```

My first method to saving the state was to get position from Leaflet map whenever the map was moved. Leaflet has some built in event handlers including the 'moveend' one above. Here I am saving the position from Leaflet in my React(Redux) state at the end of each move. The Redux state is then passed into the Map component as. This runs into issues as the position of the map is trying to be maintained in two places. The Redux state and Leaflet state can get out of synch causing strange map movement, repeated re-renderings, and crashing of the web app.

```
componentWillUnmount() {
    const leafletMap = this.leafletMap.leafletElement;

    //save current position of map
    this.props.saveMapPosition(leafletMap.getCenter(), leafletMap.getZoom());
  }
```

Instead of saving the position in Redux after every move, I moved that function to the componentWillUnmount lifecycle method. This makes a lot more sense when you look at the original problem we were trying to solve. I needed to save the position of the Leaflet map when I navigated away from the map page. The componentWillUnmount lifecycle method runs whenever this would happen and saves the map position in Redux.

The Redux state feeds to the Leaflet Map component through `this.props.map` so when the Map is reopened, the "initial state" is now the last map position.

