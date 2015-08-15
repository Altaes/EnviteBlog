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
 
 There's currently 1 document in the database as a test event, and that test information is stored in the db and the image is stored elsewhere.
 
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

**Event Feed**

Currently I have a huge problem in my event feed: The events are not loading on startup. I went back and searched online for TableViewController(TVC) behaviours and found out that before loading, the TVC would call `- (NSInteger)tableView:(UITableView *)tableView numberOfRowsInSection:(NSInteger)section` to relay the number of cells to `- (UITableViewCell *)tableView:(UITableView *)tableView cellForRowAtIndexPath:(NSIndexPath *)indexPath` which would in turn create the cells. By doing a little debugging (Ok, not a whole lot, just adding a single breakpoint), I found out that the first mentioned TVC method would return 0 as the number of cells to create. Oh, that must explain why there's no loaded event cells; well that's partially true. In the meantime, the event feed would just show this infinite running activity indicator.



How to drive away all your users 101: replace home page with just an infinite running activity indicator.



It seems to me that at the point that the TVC is getting ready to load its subviews, or in this case determining and creating the cells to be displayed, that the event data request has not been fully completed by the Data Handler. In order to further tet my hypothesis I decided to check if the TVC has any means of actually knowing when the event data has been successfully loaded by the Data Hander, and to my embarassment there was no communication between the Data Handler and my TVC. So I went back and decided to make my TVC the delegate for my Data Handler and implemented `-(void)didFinishLoadingEventData` the callback method in my TVC and set up the protocols respectively. Ok, I am now calling my delegate method whenever the Data Handler has completely finished loading the event data, this should work right? Well, as in most cases, there's always something that goes wrong, and my case was no different.



Please make it stop. (I'm not even going to give you a different screenshot)


After searching online, most notably stack overflow, I reaized that my Data Handler was running its NSURLConnection requests on a different thread, as it is a NSOperation. I also found another key piece of information that would help me greatly: The View Controllers are always ran on the main thread and I was trying to call the delegate method to reload the table view data on the secondary thread. This was a big no no, because not only did I not actually reload the data, I was stuck with that horrid activity indicator with no loaded events. So I decided to dispatch a request to call the delegate method in the main thread:

````
dispatch_async(dispatch_get_main_queue(), ^{
 if (self.delegate) {
  [self.delegate didFinishLoadingData];
 }
});
````
