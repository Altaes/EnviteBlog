# EnviteBlog
A blog that tracks my progress on the iOS App, Envite, that I'm currently developing.

# What is Envite[not-sic]?
 Envite is an event driven networking app directed at college students and small to medium sized communities. Many of times we have this internal urge to just get out there and do something with our peers, and yet many a times we simply can't find a good event to go to. Here is where Envite steps into the picture, an app that allows peers around you to submit categorized events so you don't have to scourage Facebook for something to do. Are you part of a club? Great! You can submit your event through the app, and you'll be guaranteed free publicity. Do you just want to throw down a LAN session with your peers? Awesome! You too, can freely publicize your event so EVERYONE can see the great LAN you throw. Do you just want to sponantaneously hang with your fellow peers at the beach? Gnarly! You can also publicize your event.

Envite is about all inclusivity, from event organizers to club activities to the dude who just wants to do stuff. It is about publicizing activities which we might've never heard of if it were posted on Facebook.

# Progression of Envite

NOTE: I have started writing this blog mid-development of Envite. The structure of this blog will be clustered in sub-sections (i.e. LoginView, SubmittingEvents, Server-App Interactions, etc).

<hr>
####`August 9, 2015`
<hr>
  **LoginView**
  
 The current login view looks pretty plain, and not particularly eye-catching and attention grabbing, however at least it's      not eye gorging worthy... hopefully. In the future, I hope to also provide my own login form with a email/password textfields and store user information in a separate compartment of my db.
 
 <p align="center">
 <img src="https://github.com/Altaes/EnviteBlog/blob/master/LoginScreen-8-09-15.png" height="400px" />
 </p>
 
 **Event Feed**
 
 This is the current landing page of the event feed, where users will be able to see events that they put up and as well as those of others.
 
 <p align="center">
 <img src="https://github.com/Altaes/EnviteBlog/blob/master/eventFeed-8-09-15.png" height="400px" />
 </p>
 
 There's currently 1 document in the database as a test event, and that test information is stored in the db and the image is stored elsewhere. There's also a huge problem with the loading of events on startup, the screenshot that you see here is when I click on another tab, let's say "My Events", and then click back to "Event Feed". The events only show up if you follow the aformentioned clicking process, which is seriously a huge UX problem that I need to solve.
 
 The current save button looks really plain, and I hope to make that button a little more aesthetically pleasing, but I'll save that for later because it's just a minor fix. The current auto-layout for this TableViewCell needs to be set, and there's a weird spacing on the right side of the table view cell that needs to be gotten rid of. There also needs to be 2 drop downs where the navigation bar is, one of the event search radius from user location, and another for categories that the events are placed into.
 
 The TabBar icons look slightly too large, and close to the top of the tabbar itself. There's an ongoing bug with the interface builder and setting selected tab bar icons and unselected tabbar icons, but I won't get into that. The only way was to set the icons programatically and the spacing was a little off.
 
 **User created events (Tab to manage user created events, or user moderator ones)**
 
 <p align="center">
 <img src="https://github.com/Altaes/EnviteBlog/blob/master/MyEvents-8-09-15.png" height="400px" />
 </p>
 
 There's a bunch of placeholder table view cells so I can get a feel for what the user might want or for me to get an idea on how to further make the process of managing user created events simple yet dutiful.
 
 On this view controller, the "Create an Event" View controller can be modally presented:
 
 <p align="center">
 <img src="https://github.com/Altaes/EnviteBlog/blob/master/CreateEventPresentation-8-09-15.gif" height="400px" />
 </p>
 
 The form works great, but there's still some awkwardness to it design wise (well that's what I believe). I still need to add the function of a user dragging and dropping a pin via the Map Kit, and then updating the textfield with the human address. I imagine the process is just a simple reverse geocoding, where I first get the MKLocation and then generate a top 10 or so results of addresses and return the top 1, and replace the textfield with that address.
 
 Currently the user can just select any image, or take one from the camera and that raw image is then stored on the serverside, however that's largely inefficient especially space wise. So a next improvement for the form is to reduce the resolution of the image and maybe store a thumbnail version and a moderate more high definition version, however taking much less space.
 
 **Settings page for the user**
 
 <p align="center">
 <img src="https://github.com/Altaes/EnviteBlog/blob/master/Settings-8-09-15.png" height="400px" />
 </p>
 
 This page is non-existent and I haven't thought about any extra user tools to put here, other than logging out, deleting account, sharing to friends, etc. I could put default search radius for events here, and maybe some other preferences to optimize searching, however that's up for debate.
 
 **Server sided hosting**
 
 I've written the server in Node.js, with express and mongodb, and I'm hosting it currently on AWS EC2, well through Elastic Beanstalk: Amazon's PAAS (Platform As A Service). Everything is working great on this end, but I still need to figure out a way to fast cache images (Looking at you SDWebImage) and getting an URL to a file on the EC2 instance. If I can get that working, then everything will go great, however I'm also looking at setting up a bucket in S3 for images and then implementing AWS SDK in my iOS app (hopefully I won't have to do that, and I can get a direct URL to the image successfully).
 
 **What Is Next?**
 
 I need to figure out how to background download images (Most likely using SDWebImage), and then create a launchscreen and have a hanging loader on the LoginView so that the event feed is fully set up before the user gets to it. Because as of now, it 50/50 works... on a good day, but it's more like 30/70 success/failure because the user reaches the event feed too early and the image hasn't been loaded from the server yet.
 
<hr>
####`August 14 2015`
<hr>

**Decisions, Decisions, Decisions**

 I've implemented SDWebImage trivially inside my table view, which they graciously provided a method for. I've decided not to have a hanging loading screen to make the user wait until the event feed has been fully loaded and set up. I've instead decided to dynamically check for when events are loaded so that the event feed will properly reload when there are new events. This is what I think is the best UX approach and if I were to plop down a loading screen I'm fearing that it'll deter users away from how slow it'll look. Hopefully I can get a 100/100 working success rate on loading the event feed on startup. Let's get coding!

**Event Feed**

 Currently I have a huge problem in my event feed: The events are not loading on startup. I went back and searched online for TableViewController(TVC) behaviours and found out that before loading, the TVC would call `- (NSInteger)tableView:(UITableView *)tableView numberOfRowsInSection:(NSInteger)section` to relay the number of cells to `- (UITableViewCell *)tableView:(UITableView *)tableView cellForRowAtIndexPath:(NSIndexPath *)indexPath` which would in turn create the cells. By doing a little debugging (Ok, not a whole lot, just adding a single breakpoint), I found out that the first mentioned TVC method would return 0 as the number of cells to create. Oh, that must explain why there's no loaded event cells; well that's partially true. 

<p align="center">
<img src="https://github.com/willtchiu/EnviteBlog/blob/master/objectArrayZero.png" height="400px" />
</p>

 In the meantime, the event feed would just show this infinite running activity indicator (How to drive away all your users 101: replace home page with just an infinite running activity indicator):

<p align="center">
<img src="https://github.com/willtchiu/EnviteBlog/blob/master/infiniteLoadingftw.png" height="400px" />
</p>

 It seems to me that at the point that the TVC is getting ready to load its subviews, or in this case determining and creating the cells to be displayed, that the event data request has not been fully completed by the Data Handler. In order to further tet my hypothesis I decided to check if the TVC has any means of actually knowing when the event data has been successfully loaded by the Data Hander, and to my embarassment there was no communication between the Data Handler and my TVC. So I went back and decided to make my TVC the delegate for my Data Handler and implemented `-(void)didFinishLoadingEventData` the callback method in my TVC and set up the protocols respectively. Ok, I am now calling my delegate method whenever the Data Handler has completely finished loading the event data, this should work right? Well, as in most cases, there's always something that goes wrong, and my case was no different.

<p align="center">
<img src="https://github.com/willtchiu/EnviteBlog/blob/master/infiniteLoadingftw.png" height="400px" />
</p>
`Please make it stop. (I'm not even going to give you a different screenshot)`

 After searching online, most notably stack overflow, I reaized that my Data Handler was running its NSURLConnection requests on a different thread, as it is a NSOperation. I also found another key piece of information that would help me greatly: The View Controllers are always ran on the main thread and I was trying to call the delegate method to reload the table view data on the secondary thread. This was a big no no, because not only did I not actually reload the data, I was stuck with that horrid activity indicator with no loaded events. So I decided to dispatch a request to call the delegate method in the main thread:
````
dispatch_async(dispatch_get_main_queue(), ^{
 if (self.delegate) {
  [self.delegate didFinishLoadingData];
 }
});
````
And when I ran my app again, it worked!

<p align="center">
<img src="https://github.com/willtchiu/EnviteBlog/blob/master/EventFeedLoadOnStartup.gif" height="400px" />
</p>

**At the end of the day..**

 This was definitely one of my more challenging problems, so far (Still have to get to triple-querying on the server-side, oh goodness). It was interesting to see how iOS delegates the NSURLConnection and other NSOperations in a background thread, and how it improves application efficiency and thus become less overbearing on the user. I've also become better at delegates through this problem and learned how to properly implement a singleton model (Thank you so much CS12: Basic Data Structures). I can now see how iOS delegation is such an important concept to know and also such a powerful tool when used correctly.

**What Is Next?**

 I need to create a separate database to manage users and user information. I also need to decide whether or not I should persist events that are marked as "Join" or "Save" in the user's local directory, or if it's better to track that information in the database and then request those events when the user decides to go to "My Events". The server would then query those specific events by ID, which those listed events should be under the user's data set. 
 
 TL;DR I need to add user unique interaction and real user interaction for that matter.
 
<hr>
####`August 31 2015`
<hr>

**Persisting User Data**

 In deciding how to persist user data, two solutions come into mind. First, user data can be persisted to the server and then retrieved whenever the user logs in. However, this becomes a burden to the server during times of heavy traffic, as each time a user chooses to open an app, a request will be made to retrieve their information. Therefore, I decided that it was much better to persist the user information and any other necessary info locally in NSDirectories. This way, the server wouldn't need to handle another unecessary get request. Whenever the user decides to open the app for the first time, their user info will be persisted locally as well as any other needed information. When the user exits the app and opens it again at a later time, their information would then be loaded from the locally persisted user information.
 
 As for updating user information, such as their joined and saved events, the user data will then be persisted to the server first then saved locally. For now, the system works well, however in foresight I need to deal with users who delete the app, as that invalidates the fbAccessToken and thus creates a new user. I'm thinking of importing user data from the server on app startup to curb this problem of creating multiple documents wasting db space, however importing user data itself is another GET request. I'll leave the system as it is now, and implementing a fix for this issue in the future would be trivial.
 
 <hr>
 ####`September 10 2015`
 <hr>
 
 **Personalizing User Experience (The struggle with JSON.parse() and Mongodb JSON)**
 
 At the very beginning I plastered on the 'Join' and 'Save' buttons to each event cell, and I've been meaning to get to implement their functionality since. I decided to put that on hold until I implemented the user persisting. In examining the best design process, I decided to implement the querying of user events which the user might've already created and therefore they automatically joined. I thought that if a user creates an event, it wouldn't be a too bad assumption to say that that user was joining that event. Therefore, I focused on the creation of the querying request. In order to query from the mongodb through the server, I needed to create a JSON string that would be parsable by JSON.parse() (Which I later found out through lots of trial error and frustration that using the mongodb-extended-json parser was much better for clear reasons). In figuring out what I should query, I decided I should outline the query format and what kind of response I wanted. To keep low bandwith (not that it matters now, but in foresight of the possible future), I should only make one query request, submitting an array or collection of event IDs that I want the server to return to me, also in array format. 
 
 At first, I tried to use $in, a query selector from mongodb, and passed in an array of ObjectIDs expecting the server to return to me event objects whose ID matching to the ones in the array. Not too bad right? Well, mongodb + JSON parsing by string is such a tremendous tedious process. Consider the code below:
 ````
 //This code wouldn't parse with JSON.parse(), and also wouldn't parse with the mongodb-extended-json EJSON.parse() method.
 query = JSON.parse({"_id":{"$in":[ObjectId("55e65694db9a82b61633a1ef"),ObjectId("55e6576fdb9a82b61633a1f1"),ObjectId("55e657e6db9a82b61633a1f3"),ObjectId("55e6589bdb9a82b61633a1f5")]}});
 
 //The string should've been formatted as:
{
    "_id": {
        "$in": [
            ObjectId("55e65694db9a82b61633a1ef"),
            ObjectId("55e6576fdb9a82b61633a1f1"),
            ObjectId("55e657e6db9a82b61633a1f3"),
            ObjectId("55e6589bdb9a82b61633a1f5")
        ]
    }
}
````
 Then I decided to ssh into my mongodb server and query the documents directly with the same JSON string:
````
db.events.find(
{"_id":{"$in":[ObjectId("55e65694db9a82b61633a1ef"),ObjectId("55e6576fdb9a82b61633a1f1"),ObjectId("55e657e6db9a82b61633a1f3"),ObjectId("55e6589bdb9a82b61633a1f5")]}})
````
 And...mysteriously it works! I even tried using '$oid' instead of ObjectId(), and still the JSON parser wouldn't be able to parse my string. I think the issue is with mongoDB extended JSON parser not being able to parse strict/shell mode syntax. I'll have to look further into this and do some tests myself.

 After being able to successfully query user joined events, I linked the join and save buttons to add respective events to the user data fairly trivially. The only issue I see now is that if users join an event, they shouldn't be able to save that same event. I'll have to add that functionality next. Another problem would be if users tried to join an event which they have already joined, and in this case, I show an UIAlertView to the user. This alert proves to be important because it reduces another meaningless query to add an event that the user has already joined (Another foresight server latency reducing functionality).
 
 **What still needs to be done**
 
  Apart from the small issue with users being able to save events which they have already joined, querying by category and querying events at a certain distance from the user needs to be implemented. I'll look forward to implementing two queries, one with $geoWithin (box query) and another standard category query.
