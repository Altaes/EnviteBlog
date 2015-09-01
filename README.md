# EnviteBlog
A blog that tracks my progress on the iOS App, Envite, that I'm currently developing.

# What is Envite[not-sic]?
 Envite is an event driven networking app directed at college students and small to medium sized communities. Many a times we have this internal urge to just get out there and do something with our peers, and yet many a times we simply can't find a good event to go to. Here is where Envite steps into the picture, an app that allows peers around you to submit categorized events so you don't have to scourage Facebook for something to do. Are you part of a club? Great! You can submit your event through the app, and you'll be guaranteed free publicity. Do you just want to throw down a LAN session with your peers? Awesome! You too, can freely publicize your event so EVERYONE can see the great LAN you throw. Do you just want to sponantaneously hang with your fellow peers at the beach? Gnarly! You can also publicize your event.

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
