project_path: /web/_project.yaml
book_path: /web/fundamentals/_book.yaml
description: Lorem ipsum dolor sit amet, consectetur adipiscing elit. Etiam ut tellus sit amet elit ultricies malesuada. Vestibulum consequat et ex ut mollis. Aliquam et malesuada ante. Phasellus ac tincidunt elit, at cursus mi. Aenean orci nulla, dictum non dapibus sed, ultricies sit amet purus. Sed quis turpis velit. Phasellus mollis ultrices iaculis.

{# wf_published_on: 2017-04-03 #}
{# wf_updated_on: 2017-03-14 #}

# [WIP] Mobile Web Video Playback {: .page-title }

{% include "web/_shared/contributors/beaufortfrancois.html" %}

How do you create the best mobile media experience on the Web? Easy! It all
depends on user engagement and the importance you give to the media on a web
page. I think we all agree that a web page where video is THE reason of user's
visit will have to feature an immersive and re-engaging user experience.

The goal here is to show you how to enhance in progressive way your media
experience and make it more immersive thanks to a plethora of Web APIs. That's
why we're going to build a simple mobile player experience with custom
controls, fullscreen, and background playback.


## Custom controls [50%]

### HTML layout

As you can see below, the HTML layout is pretty simple: a `<div>` root element
that contains the `<video>` element and another `<div>` element dedicated to
video controls. It contains the play/pause button, the fullscreen button, two
buttons to seek backward and forward, and some elements for current time,
duration and time tracking.

    <div id="videoContainer">
      <video id="video" src="file.mp4">
      <div id="videoControls">
        <button id="playPauseButton"></button>
        <button id="fullscreenButton"></button>
        <button id="seekForwardButton"></button>
        <button id="seekBackwardButton"></button>
        <div id="videoCurrentTime"></div>
        <div id="videoDuration"></div>
        <div id="videoTimeTrack"></div>
      </div>
    </div>

### Video metadata

Let's start simple:

    video.addEventListener('loadedmetadata', function() {
      videoDuration.textContent = formatTime(video.duration);
      videoCurrentTime.textContent = formatTime(video.currentTime);
      videoTimeTrack.style.width = `${video.currentTime * 100 / video.duration}%`;
    });

The `formatTime` function is a simple utility function that formats time in 00:00:00.

### Play/pause video

Then, let's code the play/pause button actions:

    playPauseButton.addEventListener('click', function(event) {
      event.stopPropagation();
      if (video.paused) {
        video.play();
      } else {
        video.pause();
      }
    });

Let's adjust our controls based on play and paused video events:

    video.addEventListener('play', function() {
      playPauseButton.classList.toggle('paused', true);
      updateTimeTracking();
      hideMyVideoControls();
    });

    video.addEventListener('pause', function() {
      playPauseButton.classList.toggle('paused', false);
      showMyVideoControls();
    });

    function updateTimeTracking() {
      requestAnimationFrame(function() {
        if (!video.paused) {
          // Let's continously update time tracking when video is playing.
          updateTimeTracking();
        }
        videoCurrentTime.textContent = formatTime(video.currentTime);
        videoTimeTrack.style.width = `${video.currentTime * 100 / video.duration}%`;
      });
    }

### Video ends

What happens when video ends?

    video.addEventListener('ended', function() {
      playPauseButton.classList.toggle('paused', false);
      showMyVideoControls();
    });

### Seek backward and forward

And seek backward and seek forward buttons now!

    const skipTimeInSeconds = 10;

    seekForwardButton.addEventListener('click', function(event) {
      event.stopPropagation();
      video.currentTime = Math.min(video.currentTime + skipTimeInSeconds, video.duration);
    });

    seekBackwardButton.addEventListener('click', function(event) {
      event.stopPropagation();
      video.currentTime = Math.max(video.currentTime - skipTimeInSeconds, 0);
    });

    video.addEventListener('seeking', function() {
      video.classList.toggle('seeking', true);
    });

    video.addEventListener('seeked', function() {
      video.classList.toggle('seeking', false);
      updateTimeTracking();
    });

<video controls muted playsinline>
  <source src="/web/fundamentals/getting-started/primers/videos/perfect-fullscreen.webm"
          type="video/webm; codecs=vp8">
</video>

## Fullscreen [90%]

Here we are going to take advantage of several Web APIs to create a perfect
and seamless fullscreen experience. To see it in action, check out
the final [fullscreen sample]{: .external }.

Obviously, you don't have to use all of them. Just pick the ones that make
sense to you and combine them to create your custom flow.

<video controls muted playsinline>
  <source src="/web/fundamentals/getting-started/primers/videos/perfect-fullscreen.webm"
          type="video/webm; codecs=vp8">
</video>

### Prevent automatic fullscreen

On iOS, `video` elements automagically enter fullscreen mode when media
playback begins. As we're trying to tailor and control as much as possible our
media experience across mobile browsers, I recommend you set the `playsinline`
attribute of the `video` element to force it to play inline on iPhone and not
enter fullscreen mode when playback begins.

<pre class="prettyprint lang-html">
&lt;div id="videoContainer"&gt;
  &lt;video id="video" src="file.mp4" <strong>playsinline</strong>&gt;&lt;/video&gt;
  &lt;div id="videoControls"&gt;...&lt;/div&gt;
&lt;/div&gt;
</pre>

Caution: Set `playsinline` only if you provide your own media controls or show
native controls with `<video controls>`.

<div class="clearfix"></div>
<div class="attempt-left">
  <figure>
    <img src="/web/fundamentals/getting-started/primers/videos/without-playsinline.gif"
         alt="Playback without playsinline on iPhone">
    <figcaption>
      Playback <a href="#TODO" class="external" >without <code>playsinline</code></a>
    </figcaption>
  </figure>
</div>
<div class="attempt-right">
  <figure>
    <img src="/web/fundamentals/getting-started/primers/videos/without-playsinline.gif"
         alt="Playback with playsinline on iPhone">
    <figcaption>
      Playback <a href="#TODO" class="external" >with <code>playsinline</code></a>
    </figcaption>
  </figure>
</div>
<div class="clearfix"></div>

### Toggle fullscreen on button click

Now that we prevent automatic fullscreen, we need to handle ourselves the
fullscreen mode for the video with the [Fullscreen API]{: .external}. When user
clicks the "fullscreen button", let's exit fullscreen mode with
`document.exitFullscreen()` if fullscren mode is currently in use by the
document. Otherwise, request fullscreen on the video container with the
method `requestFullscreen()` if available or fallback to
`webkitEnterFullscreen()` on the video element only on iOS.

Note: I'm going to use a [tiny shim] for the Fullscren API in code snippets below that
will take care of prefixes as the API is not unprefixed yet at that time.

    fullscreenButton.addEventListener('click', function(event) {
      event.stopPropagation();
      if (document.fullscreenElement) {
        document.exitFullscreen();
      } else {
        requestFullscreenVideo();
      }
    });

    function requestFullscreenVideo() {
      if (videoContainer.requestFullscreen) {
        videoContainer.requestFullscreen();
      } else {
        video.webkitEnterFullscreen();
      }
    }

    document.addEventListener('fullscreenchange', function() {
      fullscreenButton.classList.toggle('active', document.fullscreenElement);
    });

### Toggle fullscreen on screen orientation change

As user rotates device in landscape mode, let be smart about this and
automatically request fullscreen to create an immersive experience. For this,
we'll need the [Screen Orientation API]{: .external} which is not yet supported
everywhere.  Thus, this will be our first progressive enhancement.

How doest this work? As soon as we detect the screen orientation changes, let's
request fullscreen if the browser window is in landscape mode (that is, its
width is greater than its height). If not, let's exit fullscreen. That's all.

Note: This may silently fail in browsers that don't [allow requesting
fullscreen from the orientation change
event](https://github.com/whatwg/fullscreen/commit/e5e96a9).

    if ('orientation' in screen) {
      screen.orientation.addEventListener('change', function() {
        // Let's request fullscreen if user switches device in landscape mode.
        if (screen.orientation.type.startsWith('landscape')) {
          requestFullscreenVideo();
        } else if (document.fullscreenElement) {
          document.exitFullscreen();
        }
      });
    }

<video controls muted playsinline>
  <source src="/web/fundamentals/getting-started/primers/videos/perfect-fullscreen.webm"
          type="video/webm; codecs=vp8">
</video>

### Lock screen in landscape on button click

As video may be better viewed in landscape mode, we may want to lock screen in
landscape when user clicks the "fullscreen button". We're going to combine the
previously used [Screen Orientation API]{: .external } and some [media
queries]{: .external } to make sure this experience is the best.

Locking screen in landscape is as easy as calling
`screen.orientation.lock('landscape')`. However, we should do this only when
device is in portrait mode with `matchMedia('(orientation: portrait)')` and can
be held in one hand with `matchMedia('(max-device-width: 768px)')` as this
wouldn't be a great experience for users on tablet.

<pre class="prettyprint">
fullscreenButton.addEventListener('click', function(event) {
  event.stopPropagation();
  if (document.fullscreenElement) {
    document.exitFullscreen();
  } else {
    requestFullscreenVideo();
    <strong>lockScreenInLandscape();</strong>
  }
});
</pre>

    function lockScreenInLandscape() {
      if (!('orientation' in screen)) {
        return;
      }
      // Let's force landscape mode only if device is in portrait mode and can be held in one hand.
      if (matchMedia('(orientation: portrait) and (max-device-width: 768px)').matches) {
        screen.orientation.lock('landscape');
      }
    }

<video controls muted playsinline>
  <source src="/web/fundamentals/getting-started/primers/videos/perfect-fullscreen.webm"
          type="video/webm; codecs=vp8">
</video>

### Unlock screen on device orientation change

You may have noticed the lock screen experience we've just created isn't
perfect though as we don't receive screen orientation changes when screen is locked.

In order to fix this, let's use the [Device Orientation API]{: .external } if
available. This API provides information from the hardware measuring a device's
position and motion in space: gyroscope and digital compass for its
orientation, and accelerometer for its velocity. When we detect a device
orientation change, let's unlock screen with `screen.orientation.unlock()` if
user holds device in portrait mode and screen is locked in landscape mode.

<pre class="prettyprint">
function lockScreenInLandscape() {
  if (!('orientation' in screen)) {
    return;
  }
  // Let's force landscape mode only if device is in portrait mode and can be held in one hand.
  if (matchMedia('(orientation: portrait) and (max-device-width: 768px)').matches) {
    screen.orientation.lock('landscape')
    <strong>.then(function() {
      listenToDeviceOrientationChanges();
    })</strong>;
  }
}
</pre>

    function listenToDeviceOrientationChanges() {
      if (!('DeviceOrientationEvent' in window)) {
        return;
      }
      var previousDeviceOrientation, currentDeviceOrientation;
      window.addEventListener('deviceorientation', function onDeviceOrientationChange(event) {
        // event.beta represents a front to back motion of the device and
        // event.gamma a left to right motion.
        if (Math.abs(event.gamma) > 10 || Math.abs(event.beta) < 10) {
          previousDeviceOrientation = currentDeviceOrientation;
          currentDeviceOrientation = 'landscape';
          return;
        }
        if (Math.abs(event.gamma) < 10 || Math.abs(event.beta) > 10) {
          previousDeviceOrientation = currentDeviceOrientation;
          // When device is rotated back to portrait, let's unlock screen orientation.
          if (previousDeviceOrientation == 'landscape') {
            screen.orientation.unlock();
            window.removeEventListener('deviceorientation', onDeviceOrientationChange);
          }
        }
      });
    }

As you can see, this is the seamless fullscreen experience we were looking for.
To see this in action, check out the [fullscreen sample]{: .external }.

<video controls muted playsinline>
  <source src="/web/fundamentals/getting-started/primers/videos/perfect-fullscreen.webm"
          type="video/webm; codecs=vp8">
</video>

## Background playback [50%]

When you detect a web page or a video in the web page is not visible anymore,
you may want to update your analytics to reflect this. This could also affect
the current playback as in picking a different track, pause it, or even show
custom buttons to the user for instance.

With the Media Session API, you can also customize media notifications
by providing metadata for the currently playing video. It also allows you
to handle media related events such as seeking or track changing which may come
from notifications or media keys. To see this in action, check out the final
[background playback sample]{: .external }.

### Pause video on page visibity change

With the [Page Visibility API], we can determine the current visibility of a
page and be notified of visibility changes. Code below pauses video when page
is hidden. This happens when you switch tabs or screen lock is active for
instance.

Note: Chrome for Android already pauses videos when page is hidden.

    document.addEventListener('visibilitychange', function() {
      // Pause video when page is hidden.
      if (document.hidden) {
        video.pause();
      }
    });

### Show/hide mute button on video visibility change

If you use the new [Intersection Observer API], you can be even more granular
at no cost. This API lets you know when an observed element enters or exits the
browser's viewport.

Let's show/hide a mute button based on the video visibility in the page. If
video is playing but not currently visible, a mini mute button will be shown in
the bottom right corner of the page to give user control over video sound.

    <button id="muteButton"></button>

<div class="clearfix"></div>

    if ('IntersectionObserver' in window) {
      // Show/hide mute button based on video visibility in the page.
      function onIntersection(entries) {
        entries.forEach(function(entry) {
          muteButton.hidden = video.paused || entry.isIntersecting;
        });
      }
      var observer = new IntersectionObserver(onIntersection);
      observer.observe(video);
    }

    muteButton.addEventListener('click', function() {
      // Mute/unmute video on button click.
      video.muted = !video.muted;
    });

<video controls muted playsinline>
  <source src="/web/fundamentals/getting-started/primers/videos/video-visibility.webm"
          type="video/webm; codecs=vp8">
</video>

### Customize Media Notifications

When your web app is playing audio or video, you can already see a media
notification sitting in the notification tray. On Android, Chrome does its best
to show appropriate information by using the document's title and the largest
icon image it can find.

<div class="clearfix"></div>
<div class="attempt-left">
  <figure>
    <img src="/web/updates/images/2017/02/without-media-session.png"
         alt="Without media session">
    <figcaption>Without media session</figcaption>
  </figure>
</div>
<div class="attempt-right">
  <figure>
    <img src="/web/updates/images/2017/02/with-media-session.png"
         alt="With media session">
    <figcaption>With media session</figcaption>
  </figure>
</div>
<div class="clearfix"></div>

Let's see how to customize this media notification by setting some media
session metadata such as the title, artist, album name, and artwork with the
Media Session API.

TODO

<pre class="prettyprint">
playPauseButton.addEventListener('click', function() {
  if (video.paused) {
    video.play()
    <strong>.then(function() {
      setMediaSession();
    });</strong>
  } else {
    video.pause();
  }
});
</pre>

    function setMediaSession() {
      if (!('mediaSession' in navigator)) {
        return;
      }
      navigator.mediaSession.metadata = new MediaMetadata({
        title: 'Never Gonna Give You Up',
        artist: 'Rick Astley',
        album: 'Whenever You Need Somebody',
        artwork: [
          { src: 'https://dummyimage.com/96x96',   sizes: '96x96',   type: 'image/png' },
          { src: 'https://dummyimage.com/128x128', sizes: '128x128', type: 'image/png' },
          { src: 'https://dummyimage.com/192x192', sizes: '192x192', type: 'image/png' },
          { src: 'https://dummyimage.com/256x256', sizes: '256x256', type: 'image/png' },
          { src: 'https://dummyimage.com/384x384', sizes: '384x384', type: 'image/png' },
          { src: 'https://dummyimage.com/512x512', sizes: '512x512', type: 'image/png' },
        ]
      });
    }

Once playback is done, you don't have to "release" the media session as the
notification will automatically disappear. Keep in mind that current
`navigator.mediaSession.metadata` will be used when any playback starts. This
is why you need to update it to make sure you're always showing relevant
information in the media notification.

### Handle playlists

If your web app provides a playlist, you may want to allow the user to navigate
through your playlist directly from the media notification with some "Previous
Track" and "Next Track" icons.

    if ('mediaSession' in navigator) {
      navigator.mediaSession.setActionHandler('previoustrack', function() {
        // User clicked "Previous Track" media notification icon.
        playPreviousVideo(); // load and play previous video
      });
      navigator.mediaSession.setActionHandler('nexttrack', function() {
        // User clicked "Next Track" media notification icon.
        playNextVideo(); // load and play next video
      });
    }

Note that media action handlers will persist. This is very similar to the event
listener pattern except that handling an event means that the browser stops
doing any default behaviour and uses this as a signal that your web app
supports the media action. Hence, media action controls won't be shown unless
you set the proper action handler.

By the way, unsetting a media action handler is as easy as assigning it to `null`.

The Media Session API allows you to show "Seek Backward" and "Seek Forward"
media notification icons if you want to control the amount of time skipped.

    if ('mediaSession' in navigator) {
      let skipTime = 10; // Time to skip in seconds

      navigator.mediaSession.setActionHandler('seekbackward', function() {
        // User clicked "Seek Backward" media notification icon.
        video.currentTime = Math.max(video.currentTime - skipTime, 0);
      });
      navigator.mediaSession.setActionHandler('seekforward', function() {
        // User clicked "Seek Forward" media notification icon.
        video.currentTime = Math.min(video.currentTime + skipTime, video.duration);
      });
    }

The "Play/Pause" icon is always shown in the media notification and the related
events are handled automatically by the browser. If for some reason the default
behaviour doesn't work out, you can still [handle "Play" and "Pause" media
events].

The cool thing about the Media Session API is that the notification tray is not
the only place where media metadata and controls are visible. The media
notification is synced automagically to any paired wearable device. And it also
shows up on lock screens.

<div class="clearfix"></div>
<div class="attempt-left">
  <figure>
    <img src="/web/updates/images/2017/02/lock-screen.png" alt="Lock Screen">
    <figcaption>
      Lock Screen -
      <a href="https://wikipedia.org/wiki/Rick_Astley#/media/File:Rick_Astley_Tivoli_Gardens.jpg">
        Photo
      </a>
      by Michael Alø-Nielsen /
      <a href="https://creativecommons.org/licenses/by/2.0/">
        CC BY 2.0
      </a>
    </figcaption>
  </figure>
</div>
<div class="attempt-right">
  <figure>
    <img src="/web/updates/images/2017/02/wear.png" alt="Wear Notification">
    <figcaption style="text-align: center">Wear Notification</figcaption>
  </figure>
</div>
<div class="clearfix"></div>

Lorem ipsum dolor sit amet, consectetur adipiscing elit. Etiam ut tellus sit
amet elit ultricies malesuada. Vestibulum consequat et ex ut mollis. Aliquam et
malesuada ante. Phasellus ac tincidunt elit, at cursus mi. Aenean orci nulla,
dictum non dapibus sed, ultricies sit amet purus. Sed quis turpis velit.
Phasellus mollis ultrices iaculis.

[tiny shim]: https://github.com/beaufortfrancois/sandbox/blob/gh-pages/media/tiny-fullscreen-shim.js
[handle "Play" and "Pause" media events]: /web/updates/2017/02/media-session#play_pause
[Fullscreen API]: https://fullscreen.spec.whatwg.org/
[Screen Orientation API]: https://w3c.github.io/screen-orientation/
[media queries]: https://developer.mozilla.org/en-US/docs/Web/CSS/Media_Queries/Using_media_queries
[Device Orientation API]: https://w3c.github.io/deviceorientation/spec-source-orientation.html
[fullscreen sample]: #TODO
[background playback sample]: #TODO
[Page Visibility API]: https://www.w3.org/TR/page-visibility/
[Intersection Observer API]: /web/updates/2016/04/intersectionobserver