---
layout: post
title:      "React Router Explained My Way"
date:       2020-08-14 19:50:49 -0400
permalink:  react_router_explained_my_way
---

In my final project, I was required to create at least three routes and I exceeded that with seven. I utilized react router to create and navigate to these various routes in my application. The routing library proved to be intuitively easy to implement and absolutely imperative in creating a seamlessly navigational user experience, but by the same token, it was definitely a struggle to fully understand. Just when I thought I understood it, it turns out I fundamentally didn’t. I learned through the words of Brad Westfall—who wrote this very useful [article](https://css-tricks.com/react-router-4/) on react router by the way—that strategic placement was the name-of-the-game when it came to clearing up misunderstanding of react router as it concretely taught me what I was exactly doing wrong and why my routes weren’t working. Throughtout this blog post, I will be breaking down and explaining all the navigational components I composed to build my snappy single-page application. Also note that, in order for these components (or react router for that matter) to work, you’ll need to place `<BrowserRouter>` at the root of the application. What `<BrowserRouter>` does is it allows react router to pass the app’s routing information down to any child component it needs (via context). I will be illustrating just how powerfully illuminating strategic placement is with code examples from my project. Before I delve into it though, I would like to make sure we are on the same page about what the most basic yet essential  `<Route>` component does, which is according to the official react router doc: "render some UI when its path matches the current URL". Good? good. Let's begin. 

Here are the first few routes that serve as a gateway to my app: 

```
// App.js

     <div className='app'>
        <Switch>
          <Route path='/login'>
            <Login loginUser={this.props.loginUser} />
          </Route>
          <Route path='/signup'>
            <Signup loginUser={this.props.loginUser} />
          </Route>
          <Route path='/'>
            <Home
              loginStatus={this.props.loginStatus}
              loginUser={this.props.loginUser}
              logoutUser={this.props.logoutUser}
            />
          </Route>
          <Redirect to='/login' />
        </Switch>
     </div>
```

So by default, I am at `/login` because, once again, according to the official react router doc, `<Switch>` "renders the first child `<Route>` or `<Redirect>` that matches the location." I think the switch component was, by far, the trickiest component to understand in terms of how it works, so please pay close attention to the use of it above. Without it, my pages would not be rendering in the exact way and order that I want. The best way to specifically demonstrate this is if I were to remove the switch component. This would mean that if I either decided to login from `/login` or `signup` in order to go to `/`, "weird" things would happen. When I go to `/signup`, the signup component would render fine—since it's the next route—but if I logged in from there or `/login` to visit `/`, I would get a blank page and a piece of the ui gets rendered below the login or signup component when I refresh the page on either url. Another peculiar thread that runs through these pages when I visit them is that if I hit refresh on either one of them, I would automatically get returned to `/login` and that's NOT what I want—I want to be able to still be on the same page if I refresh it. The underlying reason for all of this very curious, quirky behavior is because `<Switch>` is unique in that it renders a route **exclusively**—which means only one `<Route>` can match and render. In contrast, every `<Route>` that matches the location renders **inclusively**—which means more than one `<Route>` can match and render at the same time. This can't be overstated enough, but remembering this is key to understanding how the switch component works. 

To pivot back to the example above, the switch component is exclusively rendering `/login`—which is the first child with a path that matches the current url—and nothing else. Now when I visit `/signup` or login from `/signup` or `/login` to go to `/`, their respective components will also exclusively render because in that moment that I care about, either path is the first route that matches the url that I'm currently at. If that's still confusing, think of it as the switch component "strictly" rendering the first matching route—effectively constraining other components and preventing them from rendering—whereas without it, routes are "loosely" rendered—hence the seemingly odd behavior of other components trying to render a blank page or "leak" some ui when their paths don't even strictly match the url. That's not all to say that you can't or shouldn't be creating and navigating to routes without `<Switch>`—you can! It's just like I said: it's all a matter of strategically placing the routes in a particular way with or without the switch component, so that they work for you! In the case above, placing the route with the path that is a prefix to other paths at the end—all within a switch component—works for me. 

Fun fact: `<Redirect>` is my favorite navigational component! I think it's the most easy and self-explanatory—and not to mention fun and satisfying—component to use and understand. Since it's within my switch component, all it's doing is it only gets rendered if no other routes match first, looping back to `/login` every time. In this case though, all the other routes do match first, so it just really serves as a fallback and reminder that `/login` is the default route to go back to when all else fails to match. It's a component that I used a lot in my project as it's the primary way to programmatically navigate throughout my app without `<Link> `, and its most typical use case was to redirect the user back to `/login` when they haven't been authenticated yet. I suggest checking out Tyler McGinnis' [post](https://ui.dev/react-router-v4-programmatically-navigate/) on this for more information—he really opened my eyes to the power of `<Redirect>`'s declarative nature.

Now here are the rest of the routes when I'm logged in:

```
// Home.js

     <>
        <Header
          username={this.props.username}
          handleLogout={this.handleLogout}
          handleSearchChange={this.handleSearchChange}
          handleSubmit={this.handleSubmit}
          term={this.state.term}
          location={this.state.location}
          loading={this.props.loading}
          isFetched={this.state.isFetched}
          locationError={this.state.locationError}
        />
        <Route exact path='/'>
          <HotAndNewRestaurantListContainer />
        </Route>
        <Route path='/search'>
          <RestaurantListContainer />
        </Route>
        <Route path='/restaurants/:id'>
          <RestaurantDetailsContainer />
        </Route>
        <Route path='/written-reviews'>
          <WrittenReviewListContainer />
        </Route>
        <Route path='/saved-restaurants'>
          <SavedRestaurantListContainer />
        </Route>
     </>
```

Right off the bat, I'm at  `/` and I can see `<Header>`—which comprises a big heading of my app name "Foodcrwlr", a search bar for looking up restaurants by term and location, user dropdown menu, and a logout button. This is the only component that provides a consistent ui and where the core features stay the same across my entire app. Below it is what we will more importantly draw our attention and focus to though: the routes. Since its path exactly matches `/`, the first route is the only one that literally gets to render out its list of new and trendy restaurants in NYC. At this point, you should have put `<Header>` and `<HotAndNewRestaurantListContainer />` together as making up the home page by default. Every other route is navigated to from here on out. Often functionally confused with `<Switch>`, the `exact` prop only renders a route (and it doesn't have to be the first one) if the path *exactly* matches the current url—and the first route happens to do exactly just that. To clearly highlight the difference between the two in this scenario, let's remove the exact prop and wrap the routes in the switch component. If I try navigating to other routes, something weird that I don't want to happen happens: the url changes to match any of the paths but the home ui stays the same. Why is that happening you may ask? It's because `<Switch>` is literally rendering out its first matching route each time—`<Route path='/'><HotAndNewRestaurantListContainer /></Route>`—ignoring all the other routes.

The final navigational component that I composed for my app but is not found in any of the code examples above is `<Link>`. Like `<Redirect>`, it's also easy and self-explanatory to use and understand. It's a declarative way to navigate around an app but in a more accessible manner—simply clicking on it will lead you to a different location. For context, I mainly used it to navigate to individual restaurant pages from a list of restaurants in my project.

I would be doing this blog post on react router a disservice if I didn't bring up what the wonderfully helpful `withRouter` does and how it essentially became the "missing piece" to the puzzle of my app so to speak. `withRouter` is a function or a higher-order component that is wrapped around a component that you want to get access to `match`, `location`, and `history` props with—it basically "injects" the component with the closest route's information. Using it in conjunction with `componentDidMount` and `componentDidUpdate` clarified and solidified my understanding of how those lifecycle methods work. Whenever I visit an individual restaurant page, its component would instantly mount with the id passed in from match params and display the correct restaurant details. No matter what page I was on, I was able to navigate back and forth with the component mounting or updating the ui to sync with the url accordingly and perfectly. It became the necessary bridge to the disconnected parts of my app where the ui was not in sync with the url. I love how it dovetailed all the routes and components to make it continuously flow from one page to the next.

Though definitely not an exhaustive and totally correct guide to using and understanding react router by any stetch of the imagination, I would say that it's personally my way of using and understanding it that can potentially be of some help or use to someone out there. 







