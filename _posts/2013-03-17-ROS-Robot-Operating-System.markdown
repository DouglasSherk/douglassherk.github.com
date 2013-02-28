---
layout: post
title: ROS, Robot Operating System
category: Coding
tags: ros c++
year: 2013
month: 03
day: 17
published: true
summary: How to set up and use ROS, because I couldn't find a good guide. Also some of my take on it.
image: post_two.png
---

<div class="row">	
	<div class="span9 columns">
		<h2>Preface</h2>
    <p>This article assumes that you already know what ROS is and the very basics about it. If you don't know, check it out at <a
    href="http://ros.org">http://ros.org</a></p>
    <p>There are already many guides that exist for ROS, explaining how to install and use it, but they're pretty bad, so I want to try to make a good one that
    has a better signal-to-noise ratio. Most of the guides are either extremely unclear or stop before explaining any functionality, API, or building
    projects.</p>
    <p>This article is fairly opinionated (at least at the beginning), based on what I think are good and bad design principles. I may also overly boil down
    concepts to make them more clear, but not entirely correct if you really get into ROS.</p>
    <h2>Disclaimer</h2>
    <p>I haven't used ROS much. The only thing I've used it for is <a href="https://anselm.ca/myhelperbot">myHelperBot</a>
    (<a href="https://github.com/DouglasSherk/myHelperBot">source code</a>), which if you go looking, you'll notice actually has no ROS code. That's because we
    had to rip it out since it was bad and wasn't working well. I've heard from several people who took the
    <a href="http://www.me.uwaterloo.ca/~me597/index.html">ME 597</a> course at UWaterloo that they spent most of their time just trying to get ROS working.
    I don't know, because I didn't take this course, but stumbled onto ROS in a different way.</p>
    <p>Part of the reason I'm writing this is because ROS is difficult to start with and get into, and I want to ease the pain for other people. I think
    despite being a fundamentally good and useful idea, the execution of it is so bad that it is difficult for me to recommend it. For this reason, I strongly
    recommend not using ROS if there are other potential solutions available to you. For example, for myHelperBot, we threw away all of our code and started
    from scratch in .NET/C#. It turned out that it ran much better, had far more documentation to get up and running with, and was much easier to set up. With
    that out of the way, let's talk about ROS.</p>
    <h2>Advantages of ROS</h2>
    <p>
      <ul>
        <li>Installed via apt-get (ideally) and thus it's pretty easy to get the basics set up (huge footnote here).*</li>
        <li>Relatively good separation of unrelated code, making ownership of responsibilities fairly clear.</li>
        <li>Great, portable, message passing IPC. The handling of message passing to Arduino is superb and very well done.</li>
        <li>Tons of example code and hack projects.</li>
        <li>Supports multiple languages running side-by-side without having to embed them in each other (C++, Python, and Lisp). Very handy.</li>
        <li>You can write your code to be highly distributed and even network it, while running the core on one master controller.</li>
      </ul>
    </p>
    <small>* while this sounds great and is indeed useful, I hear of people having to build from source quite often, especially with newer builds</small>
    <h2>Disadvantages of ROS</h2>
    <p>
      <ul>
        <li>Actually getting started is very convoluted and unnecessarily difficult.</li>
        <li>The documentation needs work, and community support is lacklustre. While it exists, most problems I had took quite a while of Googling to actually
        find a useful answer. Most of the answers you need are out there, but are drowned out by relatively useless information.</li>
        <li>Very complex and clunky for small projects, with tons of features that I'm guessing few people use.</li>
        <li>Error messages while installing and trying to run code are fairly useless.</li>
        <li>Packages (basically synonymous with a package in any package manager) take far too long to rebuild.</li>
        <li>Several executables have to be run at the same time to do anything useful. For example, to have a Kinect <=> Computer <=> Arduino interface, you
        need to run: ROS core, ROS serial Arduino helper, ROS Kinect library/helper, ROS Kinect custom code, ROS Arduino custom code.</li>
        <li>Given libraries and base code don't recover well from errors. For example, the ROS serial Arduino helper crashes if you unplug the Arduino.</li>
        <li>The concurrency model is fairly unclear, though powerful and quick to get into if you just need to get something working quickly.</li>
        <li>Example code and hack projects may or may not work. They often link to dead pages, are based on old API's with no backwards compatibility, or depend
        on hardware that you may or may not have.</li>
        <li>If you're using ROS for the Kinect functionality, don't even bother. The Microsoft Kinect SDK is so much better that it's not even worth considering
        OpenNI on ROS.</li>
      </ul>
    </p>
    <h2>Installation</h2>
    <p><small>credit in this section goes mostly to <a href="http://www.ros.org/wiki/electric/Installation/Ubuntu">the ROS website</a>; I'm largely just copying
    and pasting</small></p>
    <p>With all that out of the way, let's get started with actually installing, explaining, and then using ROS.</p>
    <p>I highly recommend using Ubuntu 11.10, regardless of the fact that it's old. I used Ubuntu 12.10 and a lot of stuff was completely broken. They may or
    may not have fixed it since then, but the only version I can vouch for is 11.10. I'm guessing 11.04 might be fine too, but wouldn't bet on it. I'm going to
    write the rest of this assuming that you're running Ubuntu 11.10, for the sake of clarity. We're also going to be setting up ROS Electric, which is one of
    the distros of ROS that I can confirm works well.</p>
    <p>Add the ROS code to your source repos:</p>
    <p><code>sudo sh -c 'echo "deb http://packages.ros.org/ros/ubuntu oneiric main" > /etc/apt/sources.list.d/ros-latest.list'</code></p>
    <p>Set up keys to authenticate the source repos with:</p>
    <p><code>wget http://packages.ros.org/ros.key -O - | sudo apt-key add -</code></p>
    <p>Update your repos:</p>
    <p><code>sudo apt-get update</code></p>
    <p>Install ROS:</p>
    <p><code>sudo apt-get install ros-electric-desktop-full</code></p>
    <p>Setup environment:</p>
    <p><code>echo "source /opt/ros/electric/setup.bash" >> ~/.bashrc</code><br />
    <code>. ~/.bashrc</code></p>
    <p>Install the rosinstall package, which lets you manage your workspace.</p>
    <p><code>sudo apt-get install python-rosinstall</code></p>
    <p>You now have everything installed, but you can't do anything useful with it yet.</p>
    <p>To verify that everything is working (or at least a good litmus test), <strong>close your current terminal and open a new one</strong>, then type:</p>
    <p><code>roscore</code></p>
    <h2>Important Concepts</h2>
    <p>I think it's best to get the installation out of the way so you can start focusing entirely on understanding things. If you haven't installed it yet, you
    should go back to the previous step and do that.</p>
    <p>
      There are some important concepts to understand about ROS before getting started.
      <ul>
        <li><em>Publisher</em>: A block of code which pushes (publishes) messages to other nodes (subscribers). Nodes will be explained later.</li>
        <li><em>Subscriber</em>: A block of code which receives (subscribes to) messages from other nodes (publishers).</li>
        <li><em>Workspace</em>: A location where all of your custom code and resources are stored.</li>
        <li><em>Package</em>: An independent set of code that can interface with other packages. It is designed to be redistributable, though you don't have to
        be careful about this, unless you want to redistribute your packages. Note that packages can refer both to "stock" code, as well as your custom code.
        Your custom packages reside within your workspace, while stock code resides in the ROS installation folder.
        <li>
          <em>Node</em>: Possibly the most useful part of ROS, nodes are an independent unit of execution, designed to operate alongside other nodes. Nodes are
          a subset of packages. That is, usually you have one or more nodes within a package. For example, a ROS package that polls a sensor for data and
          adjusts a motor based on the sensor data may be divided into:
          <ul>
            <li>Sensor node, which polls sensor data and publishes a message containing this data.</li>
            <li>Logic node, which subscribes to sensor data messages and publishes a motor power message containing a corresponding motor setting.</li>
            <li>Motor node, which subscribes to motor power messages and sets the motor power.</li>
          </ul>
          Of note here is that nodes can run on different hardware. For example, the sensor and motor nodes could run on two different Arduinos, and the logic
          node could run on a desktop, and ROS will figure this out.
        </li>
        <li><em>Launcher</em>: A tool that lets you launch several nodes at the same time, for the purposes of launching all the nodes a package needs, rather
        than forcing the user to figure out what to do with a package.</li>
      </ul>
    </p>
    <h2>What's Next</h2>
    <p>
      Now that you have everything installed, the next steps will be:
      <ol>
        <li>Create a workspace.</li>
        <li>Create a package.</li>
        <li>Create a node.</li>
        <li>Build your project (the package).</li>
        <li>Run your project. <em>After this point you can stop if you want as not everyone wants the stuff past here.</em></li>
        <li>Create another node, and edit the first one (message passing example).</li>
        <li>Create a launcher.</li>
        <li>Interface with an Arduino.</li>
        <li>Learn some useful commands.</li>
      </ol>
    </p>
    <h2>Create a Workspace</h2>
    <p>Your workspace will store everything you're working on. It can be messy (have code you're not using, etc.) and it really doesn't matter.</p>
    <p>To create:</p>
    <p><code>rosws init ~/workspace /opt/ros/electric</code></p>
    <p>The first path is, of course, wherever you want it to be. Note that it can be basically anywhere, but for the purposes of this article, I'm going to
    assume that you put it in <em>~/workspace</em>. The second path tells ROS where to look for libraries and stock code from when dealing with this
    workspace.</p>
    <p>Workspaces are otherwise not very notable. I don't recommend moving them around.</p>
    <h2>Create a Package</h2>
    <p>There's a concept related to packages called stacks, which doesn't seem very useful to me. You can read about it on the ROS site but I never needed
    them. They're a way of organizing sets of packages.</p>
    <p>To create a package, <strong>first cd into your workspace</strong>, then type:</p>
    <p><code>roscreate-pkg packagename</code></p>
    <p>To make sure it worked, type:</p>
    <p><code>rospack find packagename</code></p>
    <p>If it didn't work, that probably means that your package is not in the ROS_PACKAGE_PATH. This happened to me. I fixed it by just adding the workspace
    manually to ROS_PACKAGE_PATH in my .bashrc:</p>
    <p><code>ROS_PACKAGE_PATH=/home/user/workspace:$ROS_PACKAGE_PATH</code></p>
    <p>After adding that, type:</p>
    <p><code>source ~/.bashrc</code></p>
    <p>Then you should be good to go.</p>
    <p>We're not going to put any actual code in here, yet.</p>
    <h2>Create a Node</h2>
    <p>The node is where our actual code will go. We're going to create one in C++ for now, since for almost any project you're going to need some C++
    anyways.</p>
    <p>Navigate to <em>~/workspace/packagename/src/</em> and create a file called <em>mynode.cpp</em>. Then copy and paste the following into it:</p>
    <script src="https://gist.github.com/DouglasSherk/ad6fe8587cc7eb7ca437.js"></script>
    <p>This is basically the bare minimum that could possibly be useful in any way, and is a decent place to start for any node. This should be fairly
    self-explanatory.</p>
    <p>Open <em>CMakeLists.txt</em> and add to the very bottom:</p>
    <p><code>rosbuild_add_executable(mynode src/mynode.cpp)</code></p>
    <p>Additional nodes are added the same way. We'll cover that more in detail in another section.</p>
    <h2>Build your Package</h2>
    <p>We're ready to actually build your package. While in <em>~/workspace/packagename/</em>, type:</p>
    <p><code>rosmake</code></p>
    <p>When complete, your package will be ready to run.</p>
    <h2>Running your Project</h2>
    <p>To run this node, first you must be running the ROS core. I recommend creating two terminal tabs for this (or you could just use screen). Type:</p>
    <p><code>roscore</code></p>
    <p>You should get a welcome message. Next, type:</p>
    <p><code>rosrun packagename mynode</code></p>
    <p>Your code is now running! You can stop here or keep going. While the code is running, this is a pretty useless example, and doesn't show anything about
    ROS other than the main loop. In the next sections, we'll go over useful examples.</p>
    <h2>Create Another Node, Publisher/Subscriber</h2>
    <p>A far more useful example than the previous one is a publisher/subscriber model. To do this, we're going to edit the first node to be a publisher, and
    the second one to be a subscriber.</p>
    <p>First, rename <em>mynode.cpp</em> to <em>publisher.cpp</em> and fix it in <em>CMakeLists.txt</em>.</p>
    <p>Next, delete everything in it, and paste this into it instead.</p>
    <h4>publisher.cpp</h4>
    <script src="https://gist.github.com/DouglasSherk/1652daa0819b8baef274.js"></script>
    <p>Next, create a file called <em>subscriber.cpp</em> in the same folder, and paste this into it:</p>
    <h4>subscriber.cpp</h4>
    <script src="https://gist.github.com/DouglasSherk/82c0276432d4f1e332fd.js"></script>
    <p>Add <em>subscriber.cpp</em> to your <em>CMakeLists.txt</em>. At the end, it should now look like:</p>
    <p><pre>rosbuild_add_executable(publisher src/publisher.cpp)
rosbuild_add_executable(publisher src/publisher.cpp)</pre></p>
    <p>You can find good and more detailed instructions and explanations on the ROS website
    <a href="http://www.ros.org/wiki/ROS/Tutorials/WritingPublisherSubscriber(c%2B%2B)">here</a>.</p>
    <p>You can either run these two now, or wait until we build a launcher for them, which makes it easier to run them together. If you want to run them now,
    type in one terminal:</p>
    <p><code>rosrun packagename publisher</code></p>
    <p>Then type in another:</p>
    <p><code>rosrun packagename subscriber</code></p>
    <p>In the publisher window, you should see something like:</p>
    <p><pre>[INFO] [WallTime: 1314931831.774057] hello world 1314931831.77
[INFO] [WallTime: 1314931832.775497] hello world 1314931832.77
[INFO] [WallTime: 1314931833.778937] hello world 1314931833.78
[INFO] [WallTime: 1314931834.782059] hello world 1314931834.78
[INFO] [WallTime: 1314931835.784853] hello world 1314931835.78
[INFO] [WallTime: 1314931836.788106] hello world 1314931836.79</pre></p>
    <p>And in the subscriber window, you should see something like:</p>
    <p><pre>[INFO] [WallTime: 1314931969.258941] /subscriber_17657_1314931968795I heard hello world 1314931969.26
[INFO] [WallTime: 1314931970.262246] /subscriber_17657_1314931968795I heard hello world 1314931970.26
[INFO] [WallTime: 1314931971.266348] /subscriber_17657_1314931968795I heard hello world 1314931971.26
[INFO] [WallTime: 1314931972.270429] /subscriber_17657_1314931968795I heard hello world 1314931972.27
[INFO] [WallTime: 1314931973.274382] /subscriber_17657_1314931968795I heard hello world 1314931973.27
[INFO] [WallTime: 1314931974.277694] /subscriber_17657_1314931968795I heard hello world 1314931974.28
[INFO] [WallTime: 1314931975.283708] /subscriber_17657_1314931968795I heard hello world 1314931975.28</pre></p>
    <h2>Create a Launcher</h2>
    <p>You'll notice that the main weakness of this setup so far is that you have to have 3 terminal windows open, one for each of: the core, the subscriber,
    the publisher. Fortunately, we have an easy way to launch multiple nodes at once, called a launcher. Create a file called <em>packagename.launch</em> inside
    the <em>~/workspace/packagename/</em> folder, and paste this into it:</p>
    <h4>packagename.launch</h4>
    <script src="https://gist.github.com/DouglasSherk/eaa83054d9120e655cb9.js"></script>
    <p>Now to launch both of the nodes, you can type:</p>
    <p><code>roslaunch packagename</code></p>
    <p>Note that you still have to run roscore separately.</p>
    <h2>Interface with Arduino</h2>
    <p>One of the most useful things about ROS is its ability to relatively seamlessly interface with Arduino. This implementation even scales well, supporting
    tons of message passing that as a non-expert at serial communication I have not been able to replicate.</p>
    <p>To get started, we have to install the ROS serial package:</p>
    <p><code>sudo apt-get install ros-electric-rosserial</code></p>
    <p>With this installed, we have to copy ros_lib (a set of Arduino libraries and examples) to the Arduino sketchbook folder. The sketchbook folder is the
    location that Arduino saves to by default. If you're not sure where it is, you can also find it in the Arduino IDE's settings. You can also change it
    there. To copy ros_lib over, type the following:</p>
    <p><code>roscd rosserial_arduino/libraries</code><br />
    <code>mkdir -p sketchbook/libraries</code><br />
    <code>cp -r ros_lib sketchbook/libraries/ros_lib</code></p>
    <p>Now restart Arduino, then go to File>Examples and you should see a ros_lib folder with some examples in it.</p>
    <p>Next, we're going to create a new Arduino sketch and call it <em>publisher.pde</em>. Paste the following code into it:</p>
    <h4>publisher.pde</h4>
    <script src="https://gist.github.com/DouglasSherk/66c99d8bb35c270864aa.js"></script>
    <p>Compile and upload this to your Arduino. Once complete, make sure that roscore is running, and then run the ROS serial helper:</p>
    <p><code>rosrun rosserial_python serial_node.py /dev/ttyUSB0</code></p>
    <p>Note that the port may not be named <em>/dev/ttyUSB0</em>. You can find its name in the Arduino IDE's "Port" menu.</p>
    <p>Also note that this node will segfault whenever you unplug the Arduino. You will have to rerun it when this happens. Also, to upload code to your
    Arduino, you have to first shut this node down, then upload the code, then rerun the serial node.</p>
    <p>Finally, run the subscriber on your main computer (don't run the launcher!):</p>
    <p><code>rosrun packagename subscriber</code></p>
    <p>
      Your Arduino should now be communicating with your computer. If not, make sure the following are connected and running:
      <ul>
        <li>roscore</li>
        <li>ROS serial helper</li>
        <li>Subscriber node (on the computer)</li>
        <li>Arduino (with publisher code on it)</li>
      </ul>
    </p>
    <p>You can find more tutorials and explanations on the ROS website <a href="http://www.ros.org/wiki/rosserial_arduino/Tutorials">here</a>.</p>
    <h2>Learn some Useful Commands</h2>
    <p>
      Some useful commands that didn't make it into any of the instructions:
      <ul>
        <li><code>rostopic echo chatter</code> - echoes published messages, so you can figure out if problems are on the subscriber or publisher side (chatter
        is the topic to echo).</li>
        <li><code>roswtf</code> - displays errors and warnings about anything running attached to ROS.</li>
        <li><code>roscd</code> - cd's into a package directory, whether it's with the ROS installation or in your workspace.</li>
        <li>Find a ROS cheat sheet <a href="http://www.ros.org/wiki/Documentation?action=AttachFile&do=get&target=ROScheatsheet.pdf">here</a>.</li>
      </ul>
    </p>
    <h2>Conclusion</h2>
    <p>That's it! I also have some experience getting the Kinect stuff running, but I wouldn't recommend using it at all. If enough people want me to write a
    tutorial on that, maybe I will.</p>
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
