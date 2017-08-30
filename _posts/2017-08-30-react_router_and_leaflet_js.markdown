---
layout: post
title:  "React Router and Leaflet JS"
date:   2017-08-30 17:33:02 +0000
---


My most recent [project](https://natl-park-explorer.herokuapp.com) is a map based React application that uses the Leaflet JavaScript library. I used [react-leaflet](https://github.com/PaulLeCam/react-leaflet) to more easily integrate Leaflet into my React app. React-leaflet allows Leaflet maps to be represented as React components.

After npm/yarn installing react-leaflet and its dependencies, one simply has to import components from the library.

```
import { Map, TileLayer } from 'react-leaflet'
```

These imported components are then used like any other React component.

```
//...
render() {
  //...
  return <Map center={position}
    zoom={7}>
    <TileLayer
      url="https://api.tiles.mapbox.com/v4/mapbox.streets/{z}/{x}/{y}.png?access_token=..."
    />
    {markers}
  </Map>
}
```

To create a basic interactive map, all that is needed is the Map and TileLayer components. I used Mapbox for the source of the tileset, which gets set in the TileLayer component.

Adding interactive items to the map just requires placing them within the Map component. The {markers} in the above code is a collection of Leaflet Marker components. In my project, these were the location of U.S. National Parks.

```
import { Marker, Popup } from 'react-leaflet';

//...
return <Marker icon={greenMarker} position={position}>
      <Popup>
        <span>
          {this.props.park.fullName}
          //...
          <LinkWithContext to={`/parks/${this.props.park.id}`}>Details</LinkWithContext>
          <br/>
        </span>
      </Popup>
    </Marker>
```

Markers and Popups can be imported as components from react-leaflet. The Popup is the text box that shows when a Marker is selected.

Included in this Popup is a LinkWithContext component. This component relates to one of the difficulties I had in working with Leaflet and React. Components contained within the Leaflet map do not have access to the React router because context is not passed through to them.

To include a React router link inside the Leaflet map, the proper context must be passed in. To set the context I used the methodology suggested in [this stackoverflow](https://stackoverflow.com/questions/43465480/react-router-link-doesnt-work-with-leafletjs/43594791).

```
static contextTypes = {
  router: PropTypes.object.isRequired
}
```

To start I added this code to the component that rendered the Leaflet Map. This component encompasses the Leaflet Map so it has context to access the router. The code sets the context which is then passed into the Markers as seen below.

```
const markers = this.props.parks.map((park, index) => {
  return <ParkMarker park={park}
    key={index}
    context={this.context} />
})
```

At the Marker level, the Link to router is wrapped using a helper function prior to being rendered in our map.

```
const LinkWithContext = contextWrapper(Link, this.props.context)
```

The helper function applies the context to the Link, allowing the returned component to have access to the context we had set from outside the Leaflet map.

```
import React from 'react';
import PropTypes from 'prop-types';

function contextWrapper(WrappedComponent, context){

  class ContextProvider extends React.Component {
    getChildContext() {
      return context;
    }

    render() {
      return <WrappedComponent {...this.props} />
    }
  }

  ContextProvider.childContextTypes = {};
  Object.keys(context).forEach(key => {
    ContextProvider.childContextTypes[key] = PropTypes.any.isRequired;
  });

  return ContextProvider;
}

export default contextWrapper;
```

This takes us back to the Marker and its enclosed Popup. The Popup can link to the React router with LinkWithContext, since LinkWithContext has been wrapped with the context that was passed in from outside the Leaflet Map.

```
//...
return <Marker icon={greenMarker} position={position}>
		<Popup>
			<span>
				{this.props.park.fullName}
				//...
				<LinkWithContext to={`/parks/${this.props.park.id}`}>Details</LinkWithContext>
				<br/>
			</span>
		</Popup>
	</Marker>
```

