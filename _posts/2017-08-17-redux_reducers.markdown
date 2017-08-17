---
layout: post
title:  "Redux Reducers"
date:   2017-08-17 16:57:35 +0000
---


This post discusses the importance of how you maintain state with reducers in Redux.

My latest project has been a map based app. It pulls data from the National Park Service API to map and show information on National Parks in the United States. It uses Leaflet for the JavaScript map implementation.

![Map](https://i.imgbox.com/znvSJGnC.png)

I had been working on integrating a rating feature into the app to allow users to rate parks. The form to submit this rating was located in a popup on the map. This form submittal prompted a PATCH call to the backend API, to average the given rating into that park's overall rating. The backend would then respond with the updated park data. I would take this response from the API and update my redux state with the new park data.

```
case 'UPDATE_RATING':
  var parks = state.parks.filter((val) => {
    return val.id !== action.payload.id
  })
  return {...state, parks: [...state.parks, action.payload]}
```

Above is the general idea of how I set up my redux reducer to take in the new park data from the API response. I thought I would just filter out the old version of the updated park using filter, then add in the new one in the return statement. However, whenever I submitted a new rating, the popup would disappear, and a different park's popup would open.

After much time spent debugging I came to realization that it was an issue with this reducer. The reducer was actually changing the <em>order</em> of the parks array I had in redux. Updating the rating of a park would move that park to the end of the redux park array. This mutation of the array order resulted in the funny behavior of a different park's popup opening.

Below is an approach to updating the rating in redux that corrected this issue.

```
case 'UPDATE_RATING':
  var index = state.parks.findIndex((val) => {
    return val.id === action.payload.id
  })
  return {...state,
    parks: [...state.parks.slice(0,index),
      action.payload,
      ...state.parks.slice(index+1,state.parks.length)]
    }
```

This version replaces the park data with the new action.payload while still maintaining the order of the array. It finds the index of the park it needs to update. It then utilizes ES6 spread syntax and slicing to return the proper parks array.

