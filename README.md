# G-Tag Video Tracking Guide for VideoJS
In order to track VideoJS players with GTM on a page, we will need to add a lengthy code to the container. Loading that code on every page is not optimal and will affect the page loading speed, that’s why we need to activate that code ONLY when the Video JS player is actually embedded on that site.

## Step 1. Create a variable “Is VideoJS Player present on a page”


To do that, first, we need to create a Custom JavaScript variable and paste the following code:

```bash
function () {
 for (var e = document.getElementsByTagName("video"), x=0; x < e.length; x++) {
    if (/^blob:https?:\/\/www.bioscopelive.com/.test(e[x].src)) {
        return true;
    }
 }
 return false;
}

```

If the VideoJS player is embedded in the page, this variable will return true.

Then create a pageview trigger and use that Custom JavaScript variable in it. If the VideoJS player is present, this trigger will be activated. If there is no VideoJS player, that trigger will remain silent.


## Step 2. VideoJS Auto-Event Listener

Now, it’s VideoJS Listener’s turn. A listener is a function (or a bunch of functions) that are built to keep looking for certain interactions on a page. In this case, the listener will be looking for VideoJS player interactions. If it spots one, it will make that data visible in the Preview and Debug mode.


Create a Custom HTML tag and paste the following code.


```python
<script>

(function(dataLayer){
  var i = 0;

  // Array of percentages at which progress notifications are pushed to the dataLayer

  var markers = [10,25,50,75,90,100]; //adjust these values if you want different progress reports
  var playersMarkers = [];

  function findObjectIndexById(haystack, key, needle) {
    for (var i = 0; i < haystack.length; i++) {
      if (haystack[i][key] == needle) {
        return i;
      }
    }
    return null;
  }
  function eventToDataLayer (thisObject, eventType, currentTime) {
    var eventName;
    if (thisObject.id_) {
      eventName = thisObject.id_;
    } else {
      eventName = 'not set';
    }

    dataLayer.push({
      event: "Video",
      eventCategory: "VideoJS",
      eventAction: eventType,
      eventLabel: eventName,
      videoCurrentTime: currentTime
    });
  }

  // Loop through all Players on the page

  for (var video in window.videojs.players) {

    var player = window.videojs.players[video];

    //Pushes an object of player.id and progress markers to the array playersMarkers

    playersMarkers.push({
      'id': player.id_,
      'markers': []
    });

    player.on('play', function(e){
      var playResume = 'Resumed video';
      if (parseInt(this.currentTime()) < 2) {
        playResume = 'Played video';
      }
      eventToDataLayer (this, playResume, this.currentTime());
    });

    player.on('pause', function(e){
      eventToDataLayer (this, 'Paused video', 'Paused video');
    });

    player.on('ended', function(e){
      eventToDataLayer (this, '100%', this.currentTime());
    });

    player.on('seeking', function(e){
      eventToDataLayer (this, 'Timeline Jump', this.currentTime());
    });

    player.on('timeupdate', function(e){
      var percentPlayed = Math.floor(this.currentTime()*100/this.duration());
      var playerMarkerIndex = findObjectIndexById(playersMarkers,'id',this.id_);
      if(markers.indexOf(percentPlayed)>-1 && playersMarkers[playerMarkerIndex].markers.indexOf(percentPlayed)==-1)
      {
        playersMarkers[playerMarkerIndex].markers.push(percentPlayed);
        console.log(percentPlayed);
        eventToDataLayer (this, percentPlayed + '%', this.currentTime());
      }
    });

    player.on('error', function(e){
      eventToDataLayer (this, 'Video error', e.message);
    });

  }
})(window.dataLayer = window.dataLayer || []);

</script>
```
Don’t forget to assign the previously created Pageview Trigger:

## Checkpoint!
Let’s see what we’ve created so far:
- A Pageview Trigger which checks whether VideoJS video player is embedded in the web page (thanks to a Custom JavaScript variable).
- A VideoJS Auto-Event Listener (as a Custom HTML tag) fires only when the aforementioned Pageview Trigger activates. Every time a VideJS player interaction occurs, the listener will dispatch a Data Layer event with the following data:
  - Event Name: video (this value never changes)
  - eventCategory: VideoJS (this value never changes)
  - eventAction. Possible values: **Played, Resumed, Paused, Timeline Jump, 10%, 25%, 50%, 75%, 90%, 100%**.
  - eventLabel: **[Video title]** (Work in Progress).

If you want to test this now, enable the Preview and Debug mode, refresh the page with the VideoJS player and try interacting with it. You should start seeing video events in the Preview mode’s left side.


## Contributing
Pull requests are welcome. For major changes, please open an issue first to discuss what you would like to change.

Please make sure to update tests as appropriate.
