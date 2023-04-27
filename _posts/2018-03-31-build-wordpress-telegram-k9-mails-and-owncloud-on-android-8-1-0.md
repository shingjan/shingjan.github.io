---
title: Build Wordpress, Telegram, K9 Mails and ownCloud on Android 8.1.0
date: 2018-03-31 12:14
author: selcuk
image: 
  thumbnail: /assets/images/open_source.jpg
---
I am doing some research on the workloads of SQLite, the de facto database for Android, on mobile devices. And at the beginning, I wanted to write a wrapper for SQLiteDatabase.java and trace all relevant transactions. The amount of code refactoring is too huge for me to finish on my own. Then I tried the official SQLite android package on their website. Same problem here since all these open-source applications use SQLite in their OWN WEIRD WAYS. It takes an eternity for me to figure out how to port the package. The only way out, as I could see at that time, is to modify the Android kernel code, build it from source and flash it to my testbed, a used Nexus 6p succeeded by Xiaomi Mix, which is a great phone by the way. And that is how my last post came out.

Recently I just know the AOSP version on my Nexus 6p could see the debug log messages, while I don't think it would work on the factory image. Now I could just build those open-source applications on Android studio and see all these loggings! After all these strugglings there seems to be finally a way out. During my experimental builds on those applications I found out the setup guides for those applications are not very straightforward and decided to mark some of the tips down here.
<ol>
 	<li>Wordpress
<ol>
 	<li>Wordpress is a perfect application in terms of matching my research purposes. Clean builds, massive usages on SQLite and easy to see through. The only problem is when I try to build it with the newest version of Gradle, which is not a good idea. Except that, you would be all set.</li>
</ol>
</li>
 	<li>Telegram
<ol>
 	<li>Telegram suckssssssssssssssssssss. Since it would connect to its own server with your own build, which makes it impossible to see the SQLite trace.</li>
 	<li>Download the source: git clone --recursive-submodules https://github.com/DrKLO/Telegram.git</li>
 	<li>Get your own
<ol>
 	<li>Telegram app api id and hash <a href="https://core.telegram.org/api/obtaining_api_id">here</a>.</li>
 	<li>Google config json <a href="https://developers.google.com/mobile/add">here</a> with package name "org.telegram.messengner" and choosing "Cloud Messaging"</li>
 	<li>Hockey app id and hash <a href="https://rink.hockeyapp.net">here</a> with package name "org.telegram.messengner".</li>
 	<li>Put all of the above in TMessagesProj/src/main/java/org/telegram/messenger/BuildVars.java</li>
</ol>
</li>
 	<li>Comment
<ol>
 	<li>
<pre id="35a3" class="graf graf--pre graf-after--p">applicationIdSuffix “.beta”</pre>
</li>
 	<li>The whole signingConfigs section</li>
 	<li>
<pre id="c139" class="graf graf--pre graf-after--p">signingConfig signingConfigs.debug
signingConfig signingConfigs.release
signingConfig signingConfigs.release</pre>
</li>
 	<li>in the module-level build.gradle file. Then you will be all set.</li>
</ol>
</li>
</ol>
</li>
 	<li>K9 Mail
<ol>
 	<li>Just remember to clone recursively on all sub-modules. K9 mail is not a good target since it use SQLite in a very light way</li>
</ol>
</li>
 	<li>ownCloud
<ol>
 	<li>Same as K9-mail, Easy to make and not so many useful traces. The only thing funny is that this open-source app is actually not-free on Google play store. Touche to be such a crappy app and charge users money for using it.</li>
</ol>
</li>
</ol>
