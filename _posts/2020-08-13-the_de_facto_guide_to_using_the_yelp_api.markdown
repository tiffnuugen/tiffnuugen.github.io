---
layout: post
title:      "The De Facto Guide to Using the Yelp API"
date:       2020-08-13 16:49:31 -0400
permalink:  the_de_facto_guide_to_using_the_yelp_api
---

As I promised in my final project blog post, I will go into greater detail on how to properly consume the yelp fusion api for your final react/redux project here. Let me start from the beginning of how I approached it, work my way to all the pain points that I encountered, and show you my final solution.  

Getting access to the yelp api is really straightforward and easy as obtaining the client id and api key, then saving it in a `.env` file. After that, keep in mind that for each api call you make, your key needs to be passed as the header value like so:

```
headers: {
            Authorization: `Bearer ${process.env.REACT_APP_API_KEY}`
          }
```

Next, go into the file where your action creator lives. For me, that was in my `yelpActions.js` file within my `actions` folder. Make sure you have redux thunk installed to handle asynchronous actions before doing the following: choose an endpoint you want to get data back from and pass in the required parameters to your action creator. (NOTE: You can always use postman first to check out what each endpoint returns.) Overall, it should look something like this: 

```
// src/actions/yelpActions.js

import axios from 'axios';

export const fetchRestaurants = (searchValues) => {
  return (dispatch) => {
    dispatch({ type: 'LOADING_RESTAURANTS' });
    axios
      .get(
        `https://api.yelp.com/v3/businesses/search?categories=restaurants&location=${searchValues.location}&term=${searchValues.term}`,
        {
          headers: {
            Authorization: `Bearer ${process.env.REACT_APP_API_KEY}`
          }
        }
      )
      .then((res) =>
        dispatch({ type: 'ADD_RESTAURANTS', restaurants: res.data.businesses })
      )
      .catch((error) => console.log(error.response));
  };
};
```

As you can see, I'm using `axios` to make my api call. You can use the `fetch()` instead if you want, but I personally like to use axios because it's better at error-handling and it makes data transformation easier. 

At this point, if you also set up your reducer correctly to handle the corresponding actions that you dispatched (or sent) from your component, then you should run into a CORS error (specifically a 500 error or something like that)! I know, bummer, right? It's ok though because it turns out the yelp api doesn't support CORS. The workaround to this, though, is with [cors-anywhere](https://github.com/Rob--W/cors-anywhere). It's basically a proxy that is used to bypass the CORS restrictions. To use it, prepend the url to the yelp api endpoint like this: 

```
// src/actions/yelpActions.js

import axios from 'axios';

export const fetchRestaurants = (searchValues) => {
  const corsApiUrl = 'https://cors-anywhere.herokuapp.com/';
  return (dispatch) => {
    dispatch({ type: 'LOADING_RESTAURANTS' });
    axios
      .get(
        `${corsApiUrl}https://api.yelp.com/v3/businesses/search?categories=restaurants&location=${searchValues.location}&term=${searchValues.term}`,
        {
          headers: {
            Authorization: `Bearer ${process.env.REACT_APP_API_KEY}`
          }
        }
      )
      .then((res) =>
        dispatch({ type: 'ADD_RESTAURANTS', restaurants: res.data.businesses })
      )
      .catch((error) => console.log(error.response));
  };
};
```

GOOD NEWS: now you should be able to successfully make calls to the yelp api! BAD NEWS: you will most likely only be able to make a very limited number of calls—100 or so but feels much less than that to be honest—before you get the 429 (Too Many Requests) error.

For a month and a half or so, I was sluggishly making calls to the yelp api with the approach above and it was incredibly frustrating and annoying to hit the limit. I hated how I had to wait a while to make those calls again—it was a flow killer for sure. 

Reaching a breaking point, I realized there must be a better way to do this. There was. After furiously working the google search engine, I came across this [post](https://medium.com/@mariacristina.simoes/how-to-query-yelp-fusion-api-with-react-ruby-on-rails-15bfd04db88a) from a former flatironer and [confirmation from a yelp engineer](https://github.com/Yelp/yelp-fusion/issues/386) that it's best to just make requests to their api from the backend.

Now I present to you, my final solution: 

In your rails backend, use a HTTP client library (I used faraday below) to do what we originally did with the action creator but within your controller file. For me, that was my `yelp_controller.rb` file within my `controllers` folder. lt looks like the following: 

```
// app/controllers/yelp_controller.rb

def search
    res = Faraday.get("https://api.yelp.com/v3/businesses/search") do |req|
      req.headers['Authorization'] = "Bearer #{ENV['API_KEY']}"
      req.params['categories'] = 'food,restaurants'
      req.params['term'] = params[:term]
      req.params['location'] = params[:location]
    end
    search_results = JSON.parse(res.body)
    render json: search_results
  end
```

And here's the corresponding route: 

` post '/search', to: 'yelp#search'`

On the react/redux frontend, the action creator should now look like this: 

```
// src/actions/yelpActions.js

import axios from 'axios';

export const fetchRestaurants = (searchValues) => {
  return (dispatch) => {
    dispatch({ type: 'LOADING_RESTAURANTS' });
    axios
      .post('http://localhost:3001/search', {
        term: searchValues.term,
        location: searchValues.location
      })
      .then((res) => {
        dispatch({
          type: 'FETCH_RESTAURANTS',
          restaurants: res.data.businesses
        });
      });
  };
};
```

Basically, what I'm doing on the backend is I'm using a HTTP client library to query the yelp api with the user input (term and location) that was sent from the frontend. With everything else still in place, I was finally able to nip any and all CORS errors in the bud and merrily make all the API calls I wanted! 

Hope this helps!
 




