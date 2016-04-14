Simone Giertz fish slap
=======================

This is a reimplementation of <http://eelslap.com/>, but showing [Simone Giertz being slapped by a rubber fish][1]. Why? [Because it is a funny silly useless project][2].

[1]: https://www.instagram.com/p/6VOM75Rfzv/
[2]: https://www.youtube.com/watch?v=lrSSfd_G9gE

Online demo
-----------

**<https://denilsonsa.github.io/simone-giertz-fish-slap/fishslap_view_as_bg.html>**

Summary
-------

Just open `fishslap_view_as_bg.html` and have fun!

An alternative implementation (but otherwise identical to the user) is also available as `fishslap_view_as_canvas.html`.

If you want to generate your own preprocessed JPG file, use `fishslap_preprocess.html`. Make sure to edit the global variables at the source code (both at the *preprocess* file and at the *view* file).

Technical issues
----------------

### First try

`fishslap_v1.html` shows the simplest implementation: render a video and seek it according to the mouse position. The code couldn't be any simpler.

Unfortunately, upon trying that, I discovered that seeking in a compressed video is slow. There was a very long delay between moving the mouse and the video being updated. The experience was not smooth, it was sluggish.

### Second try

The next idea is to capture all the frames from the video, store them in a `<canvas>`, and then just show the correct frame from the `<canvas>` when needed. It would solve the delay, but requires some preprocessing. How hard can it be?

Well, `fishslap_v2.html` shows that it is harder than what I initially thought. In this file, I tried playing the video and then using `timeupdate` event to capture each frame. However, depending on the browser, [that event is only fired a few times per second][tu1], [instead of every frame][tu2]. And what if I use `requestAnimationFrame()` instead of the event? Well, it still leads to choppy capture. Not good.

In addition, browsers have a limit on how large the `<canvas>` can be, so I had to reduce the capture framerate and break the single row of frames into multiple rows (to reduce the `<canvas>` width).

And, by the way, there is no API to get the number of *frames per second* of a video. I had to get this value from elsewhere and hardcode it into the JavaScript code.

[tu1]: https://stackoverflow.com/questions/17044567/get-frame-change-in-video-html5
[tu2]: https://stackoverflow.com/questions/9678177/how-often-does-the-timeupdate-event-fire-for-an-html5-video

### Third try

Since I can't capture the frames when playing the video, my next approach is to use JavaScript to manually seek to each frame in the video, and capture it. `fishslap_v3.html` shows this solution. It also adds a progress bar while the preprocessing is happening.

I was pretty happy with results when trying this on both Chrome and Firefox on desktop. Then I added code for using the device tilt, and then I started [testing on my phone][rd]. Oh my… It didn't work at all.

I discovered that the mobile version of Chrome will not play the video unless the user actively interacts with the page to play it.

    Failed to execute 'play' on 'HTMLMediaElement': API can only be initiated by a user gesture.

Since the mobile browser will not preprocess the video automatically, I added the ugly *Preload!* button. Good, now I see the progress bar moving…

But it still didn't work. Upon further inspection, it seemed like it captured the same frame all the time. Then I tried adding a `.play()` call, and the result didn't improve.

Okay, this approach has reached a dead-end. I came to the conclusion that the preprocessing must be done on a desktop device, then saved as a single image, and then used on all devices.

[rd]: https://developer.chrome.com/devtools/docs/remote-debugging

### Fourth try

Now I have split the page into two: `fishslap_preprocess.html` for preprocessing the video and generating a JPG image file, and a `fishslap_view_as_canvas.html` to have fun playing with the fishslap.

Everything was great, except that Chrome 49 on Android decided to draw a 2×2 tiled version of the frames, instead of the expected single frame.

Then I decided to rewrite it using a simple CSS properties `background-image` and `background-size` and `background-position`. Then I discovered the bug persisted in this case as well. Well, I should report this bug.

Finally, [I discovered Chrome on Android only has this bug with the large 7680×3840 image.][bug] If I resize it to 50% (i.e. 3840×1920), then it works fine.

[bug]: https://crbug.com/603388
