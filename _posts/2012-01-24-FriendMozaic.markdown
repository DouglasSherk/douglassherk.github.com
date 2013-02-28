---
layout: post
title: FriendMozaic
category: Coding
tags: canvas facebook cors
year: 2012
month: 01
day: 24
published: true
summary: A Facebook app written at a hackathon to make a profile picture out of tiles of your friends' pictures.
image: post_one.png
---

<div class="row">	
	<div class="span9 columns">
		<h2>Preface</h2>
    <p>I recently participated in the Facebook Hackathon at the University of Waterloo. The project I chose to work on was suggested by one of my teammates, who
    showed an app for a Facebook clone in China that uses thumbnails of your friends’ profile pictures to rebuild your profile picture. We called our app
    FriendMozaic.</p>
    <p>You can find the code here: <a href="https://github.com/DouglasSherk/FriendMozaic">https://github.com/DouglasSherk/FriendMozaic</a></p>
    <p>This project got an honorable mention, which I was happy with considering most of my team had to leave just a few hours in. The project that won was
    <a href="http://ninjaquote.com">Ninjaquote</a>, which to me was basically a clinic on how to do UI design and polish things that are relatively simple
    concepts at the core.</p>
    <h2>What it Does</h2>
    <p>Back to FriendMozaic, if you don’t understand what I mean by my explanation of what it does, here’s what the result of our app’s churning looks like:</p>
    <p><img src="/img/friendmozaic.png" /></p>
    <p>This is a joke picture of me, which I used for most of my testing because of the decent contrast between different parts of the image.</p>
    <p>The user is given an option to select up to 500 of their friends to build a mosaic profile picture out of.</p>
    <p>I don’t have any information on the China Facebook clone app anymore, but when I saw it, I was instantly sold on building it for Facebook. We decided
    that, to reduce load on the server, we would do as much of it as possible in canvas/JS as opposed to PHP/GD, which is what I automatically decided would
    be the best way to do it. Also, while I generally don’t like doing things just because they’re technically more difficult (instead preferring building
    something useful and ideally simpler), we decided that it was worth going out of our comfort zone and using canvas for the purposes of the
    hackathon.</p>
    <p>I was the only one on the team who had any experience with canvas (mostly from writing conformance tests while I worked at Mozilla), so I ended up
    writing all of the canvas code, including pulling pictures from profiles and rendering them. This is what I spent most of my time on; stitching this code
    into the UI that my team built took a relatively short time afterwards.</p>
    <h2>Image Generation Algorithm</h2>
    <p>What I really want to focus on here, though, is the technical problems I ran into. Canvas is a relatively simple API, but there are some “gotcha”‘s that
    are important to see and be able to respond to.</p>
    <p>For starters, the algorithm used to actually figure out how to populate the squares of the picture was relatively simple. Basically, my algorithm
    consisted of the following:</p>
    <ul>
      <li>Load the user's profile picture.</li>
      <li>Load the profile pictures of the user’s friends that they selected.</li>
      <li>
        Redraw the canvas every time a batch of 5 friend profile pictures are successfully loaded. As part of this redrawing:
        <ul>
          <li>
            Divide the user’s profile picture into 10×10 segments.
            <ul>
              <li>
                For each 10×10 segment, get the average RGB value (i.e. add up the RGB values of each pixel, then divide by 10*10).
                <ul>
                  <li>For each friend profile picture currently loaded (or all if loading is complete), compare average RGB value to the RGB value calculated
                  in the parent step using a simple distance formula. Store this as the closest if it is.</li>
                </ul>
              </li>
              <li>Draw a friend’s profile picture which is the closest in RGB value to the 10×10 block.</li>
            </ul>
          </li>
        </ul>
      </li>
    </ul>
    <p>That’s it! We considered adding stuff like tinting so that each 10×10 block looked closer to the original, but didn’t have time to do that.</p>
    <h2>Cross-Origin Resource Sharing</h2>
    <p>Despite this simplicity, there were some other problems. The main one, which took me almost 4 hours to solve, was caused by CORS (or rather the lack of
    support for it). CORS stands for “Cross-Origin Resource Sharing”, which is a technology that allows you to bypass a security feature in the DOM which
    prevents you from copying pixel data from domains other than yours. This is done because otherwise, scripts could grab arbitrary data from other sites
    while being authenticated as the browser user (since this is not run server-side). As far as I know, this has nothing to do with the WebGL/canvas
    uninitialized memory exploit.</p>
    <p>I ran into this stuff at Mozilla but never fully understood it. The nature of conformance tests is that they’re run entirely locally, so the errors I
    ran into there were the same, but were solved in a completely different way. This time I had to fully dig into the issue and figure out what was
    happening and how to deal with it. I was surprised by how little documentation or support there is on this issue. Maybe there’s just not a lot of people
    doing cross-origin canvas work.</p>
    <p>The basic problem was that I was loading Facebook profile pictures from facebook.com, using instructions provided by friendmozaic.com, then reading the
    pixel data of these images, but the DOM security features prevented me from doing this. Fortunately, CORS provides a way around this. The basic setup of
    CORS is the following:</p>
    <ol>
      <li>
        <p>Load your image in the following way:</p>
        <script src="https://gist.github.com/DouglasSherk/16e0efd6b63c62428555.js"></script>
        <p>Note the crossOrigin parameter.</p>
      </li>
      <li>
        <p>Set up the server pointed to by http://crossdomain.com to give the following header:</p>
        <p><code>Access-Control-Allow-Origin: *</code></p>
        <p>You can also change the wildcard to a specific domain or set of domains. This can be set in a .htaccess or, for example, as part of PHP’s header()
  function.</p>
      </li>
    </ol>
    <p>More information can be found on <a href="http://hacks.mozilla.org/2011/11/using-cors-to-load-webgl-textures-from-cross-domain-images/">this
    Mozilla blog post</a> by my previous colleague and mentor, Benoit Jacob.</p>
    <p>So this looks fairly trivial, but unfortunately the second step was impossible in my case. Facebook doesn’t pass these headers as part of their
    content (despite the stuff I was grabbing being available without authentication), so I couldn’t use this method. The only other way I could think of was
    to just proxy the images.</p>
    <h2>Proxying Image Requests</h2>
    <p>What does that mean? I wrote a simple PHP script that takes in a URL parameter being the image requested, and the server would make that request then
    give it to the client as if it came from the server’s domain. Here’s what that looks like:</p>
    <script src="https://gist.github.com/DouglasSherk/be2555da6610afbdb283.js"></script>
    <p>Please do not use this! It obviously requires much more authentication and validation before it can actually be put into production code.</p>
    <p>With some polish, this is the method we currently use. It actually works fairly well, although it adds some heavy latency to the actual loading of the
    profile pictures. In general, it takes ~10 seconds for the mosaic to even begin compositing.</p>
    <p>One last sort of technical problem! Facebook is terrible at exposing some basic functionality. For example, here's a code fragment I found on Stack
    Overflow (I think) which scrapes the path to the user's profile picture, i.e. the big version:</p>
    <script src="https://gist.github.com/DouglasSherk/e37056701ec8ae66ad34.js"></script>
    <p>This could be a lot better, but I suppose it works.</p>
		<h2>Conclusion</h2>
    <p>Something I realized after I had built it in canvas was that the whole "re-render every 5 picture loads" mechanism would have been impossible or
    infeasible if I had done this in GD. Also, any interactivity, which I initially wanted to do, would have been much more difficult. It made me realize that
    I should have thought this through more at the beginning instead of just diving in. Fortunately, I made the right decision, despite initial reservations.</p>
    <p>That's it for now. I'm still tossing up whether or not I should continue this project. If I do, I'll probably rewrite the whole UI and most of the
    canvas code. I noticed a non-mispelled version of our project already exists at <a href="http://friendmosaic.com">http://friendmosaic.com</a>, which I
    actually didn't find until I registered friendmozaic.com (I was tired and actually thought that was the correct spelling). Fortunately, these guys
    seem to only work with Twitter.</p>
	</div>
</div>

<div class="row">	
	<div class="span9 column">
			<p class="pull-right">{% if page.previous.url %} <a href="{{page.previous.url}}" title="Previous Post: {{page.previous.title}}"><i class="icon-chevron-left"></i></a> 	{% endif %}   {% if page.next.url %} 	<a href="{{page.next.url}}" title="Next Post: {{page.next.title}}"><i class="icon-chevron-right"></i></a> 	{% endif %} </p>  
	</div>
</div>

<div class="row">	
  <div class="span9 columns">
    <h2>Comments Section</h2>
      <p>Feel free to comment on the post but keep it clean and on topic.</p>	
    <div id="disqus_thread"></div>
    <script type="text/javascript">
      var disqus_shortname = 'DouglasSherk'; // required: replace example with your forum shortname
      var disqus_identifier = '{{ page.url }}';
      var disqus_url = 'http://douglassherk.github.com{{ page.url }}';

      (function() {
        var dsq = document.createElement('script'); dsq.type = 'text/javascript'; dsq.async = true;
        dsq.src = 'http://' + disqus_shortname + '.disqus.com/embed.js';
        (document.getElementsByTagName('head')[0] || document.getElementsByTagName('body')[0]).appendChild(dsq);
      })();
    </script>
    <noscript>Please enable JavaScript to view the <a href="http://disqus.com/?ref_noscript">comments powered by Disqus.</a></noscript>
    <a href="http://disqus.com" class="dsq-brlink">blog comments powered by <span class="logo-disqus">Disqus</span></a>
  </div>
</div>

<!-- Twitter -->
<script>!function(d,s,id){var js,fjs=d.getElementsByTagName(s)[0];if(!d.getElementById(id)){js=d.createElement(s);js.id=id;js.src="//platform.twitter.com/widgets.js";fjs.parentNode.insertBefore(js,fjs);}}(document,"script","twitter-wjs");</script>

<!-- Google + -->
<script type="text/javascript">
  (function() {
    var po = document.createElement('script'); po.type = 'text/javascript'; po.async = true;
    po.src = 'https://apis.google.com/js/plusone.js';
    var s = document.getElementsByTagName('script')[0]; s.parentNode.insertBefore(po, s);
  })();
</script>
