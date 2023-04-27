---
title: Build AOSP 8.1.0 on Nexus 6p - An Incomplete Guide
date: 2018-03-26 05:02
author: selcuk
image: 
  thumbnail: /assets/images/aosp.jpg
---
Although Google has posted a perfect <a href="https://source.android.com/setup/initializing" target="_blank" rel="noopener">tutorial</a> on Building AOSP from the source, I decide to lay down my steps on building android on my Nexus 6p. These are a few unexpected bugs, errors and surprises as well.
<ol>
 	<li>Establishing the environment
<ol>
 	<li>64-bit Ubuntu 14.04 LTS. You can also try other ubuntu distributions with required build tools, as well as MacOS.</li>
 	<li>Installing JDK:
<ol>
 	<li><a href="http://old-releases.ubuntu.com/ubuntu/pool/universe/o/openjdk-8/openjdk-8-jre-headless_8u45-b14-1_amd64.deb">openjdk-8-jre-headless_8u45-b14-1_amd64.deb</a></li>
 	<li><a href="http://old-releases.ubuntu.com/ubuntu/pool/universe/o/openjdk-8/openjdk-8-jre_8u45-b14-1_amd64.deb">openjdk-8-jre_8u45-b14-1_amd64.deb</a></li>
 	<li><a href="http://old-releases.ubuntu.com/ubuntu/pool/universe/o/openjdk-8/openjdk-8-jdk_8u45-b14-1_amd64.deb">openjdk-8-jdk_8u45-b14-1_amd64.deb</a></li>
 	<li>download the .deb packages since OpenJDK 8 is not natively supported on Ubuntu 14.04 LTS.</li>
 	<li>sudo apt-get update</li>
 	<li>sudo dkpg -i xxx.deb | run this command on each of the above downloaded packages</li>
 	<li>if there is any missing dependency packages reported, run: sudo apt-get -f install</li>
 	<li>Repeat and loop the above two steps to make sure OpenJDK is installed.</li>
</ol>
</li>
 	<li>Installing other required packages:
<ol>
 	<li>
<pre class="devsite-terminal devsite-click-to-copy">sudo apt-get install git-core gnupg flex bison gperf build-essential zip curl zlib1g-dev gcc-multilib g++-multilib libc6-dev-i386 lib32ncurses5-dev x11proto-core-dev libx11-dev lib32z-dev ccache libgl1-mesa-dev libxml2-utils xsltproc unzip</pre>
</li>
</ol>
</li>
 	<li>Configuring USB access for your devices, in case your are not using an emulator
<ol>
 	<li>plug your android in, go to System-About Phone-Build Number, tap it for seven times and you will see the hidden developer options in the parent menu, allow USB-debugging in developer options</li>
 	<li>sudo apt-get install android-tools-adb</li>
 	<li>sudo adb devices (allow this action on your phone to get efficient permission)</li>
 	<li>change <a href="https://stackoverflow.com/questions/28704636/insufficient-permissions-for-device-in-android-studio-workspace-running-in-opens?utm_medium=organic&amp;utm_source=google_rich_qa&amp;utm_campaign=google_rich_qa">51_android_rules</a> if the above step does not work</li>
</ol>
</li>
 	<li>Using a separate output directory:
<ol>
 	<li>add the following line in your bash profile to make sure this still applies after a reboot:
<ol>
 	<li>
<pre class="devsite-terminal devsite-click-to-copy">export OUT_DIR_COMMON_BASE=&lt;path-to-your-out-directory&gt;</pre>
</li>
</ol>
</li>
</ol>
</li>
</ol>
</li>
 	<li>Downloading the source
<ol>
 	<li>Installing repo:
<ol>
 	<li>Make sure you have a bin/ directory in your home directory and that it is included in your path:
<ol>
 	<li>
<pre class="devsite-click-to-copy"><code class="devsite-terminal">mkdir ~/bin</code></pre>
</li>
 	<li>
<pre class="devsite-click-to-copy"><code class="devsite-terminal">PATH=~/bin:$PATH</code></pre>
</li>
</ol>
</li>
 	<li>Download the Repo tool and ensure that it is executable:
<ol>
 	<li>
<pre class="devsite-click-to-copy"><code class="devsite-terminal">curl https://storage.googleapis.com/git-repo-downloads/repo &gt; ~/bin/repo</code> <code class="devsite-terminal"></code></pre>
</li>
 	<li>
<pre class="devsite-click-to-copy"><code class="devsite-terminal">chmod a+x ~/bin/repo</code></pre>
&nbsp;</li>
</ol>
</li>
 	<li>Initializing a Repo client
<ol>
 	<li>Creating an working directory. This would be the directory with all source code and pre-built binaries. Name it carefully.
<ol>
 	<li>mkdir WORKING+DIR &amp;&amp; cd WORKING_DIR</li>
</ol>
</li>
 	<li>Configuring Git:
<ol>
 	<li>git config --global user.name "xxx"</li>
 	<li>git config --global user.email "xxx"</li>
</ol>
</li>
 	<li>
<pre class="devsite-terminal devsite-click-to-copy">repo init -u https://android.googlesource.com/platform/manifest</pre>
<ol>
 	<li>Download the source. REMINDER: DON'T download master branch if you do not know your phone's build number. For example, I want to build the latest stable release of Nexus 6p. The most recent build number would be android-8.1.0_r18 on 03/26/2018:
<ol>
 	<li>
<pre class="devsite-terminal devsite-click-to-copy">repo init -u https://android.googlesource.com/platform/manifest -b android-8.1.0_r18</pre>
</li>
</ol>
</li>
</ol>
</li>
 	<li>A successful initialization will end with a message stating that Repo is initialized in your working directory. Your client directory should now contain a <code>.repo</code> directory where files such as the manifest will be kept.</li>
</ol>
</li>
 	<li>Downloading the Android Source Tree by: repo sync</li>
</ol>
</li>
 	<li>Obtain proprietary binaries
<ol>
 	<li>Although Android is free and open-source, there are some hardware-related proprietary libraries required. As for a Google/Nexus phone, you can easily find those binaries <a href="https://developers.google.com/android/drivers#angleropm5.171019.017">here </a>with build number OPM5.171019.017 (android 8.1.0_r18).</li>
 	<li>Unzip those binaries, move those two shell script into your working directory and run them. After signing the agreement you would be all set.</li>
 	<li>run "make clobber" to ensure a clean source tree without other outputs</li>
</ol>
</li>
</ol>
</li>
 	<li>Building AOSP from the scratch
<ol>
 	<li>Set up the environment
<ol>
 	<li>source build/envsetup.sh</li>
 	<li>lunch</li>
 	<li>choose a target, for my case, it would be 28. angler_user_debug</li>
 	<li>make -j16, I am using a i7-7700 with 4 cores and 8 threads. Hence any argument between 16-32 should be fine for make</li>
</ol>
</li>
 	<li>Running builds on your devices
<ol>
 	<li>make adb fastboot if you haven't had them installed before</li>
 	<li>unlock your device if it is not already unlocked
<ol>
 	<li>fastboot flashing unlock</li>
</ol>
</li>
 	<li>flash the built images!
<ol>
 	<li>sudo adb reboot bootloader</li>
 	<li>fastboot flashall -w</li>
</ol>
</li>
 	<li>So far, my Nexus 6p works just fine on unmodified AOSP build version but stuck at boot screen if source code is modified even slightly.</li>
</ol>
</li>
 	<li>In case something goes wrong and there should be a backup plan:
<ol>
 	<li>Restoring devices to factory state
<ol>
 	<li>Download the factory image <a href="https://developers.google.com/android/images">here</a>.</li>
 	<li>Make sure your phone is connected by "fastboot devices"</li>
 	<li>Unzip the image and run the ./flashall.sh inside the directory</li>
</ol>
</li>
</ol>
</li>
</ol>
</li>
 	<li>Other errors:
<ol>
 	<li>Previously I was using a crappy laptop so most of the time buildings didn't go smoothly as it is now on a desktop. And below are some errors I hope you would not encounter.</li>
 	<li>Jack server out of memory error, try increase heap size:
<ol>
 	<li>
<pre class="default prettyprint prettyprinted"><code><span class="kwd">export</span><span class="pln"> JACK_SERVER_VM_ARGUMENTS</span><span class="pun">=</span><span class="str">"-Dfile.encoding=UTF-8 -XX:+TieredCompilation -Xmx4g"</span>
<span class="pun">./</span><span class="pln">prebuilts</span><span class="pun">/</span><span class="pln">sdk</span><span class="pun">/</span><span class="pln">tools</span><span class="pun">/</span><span class="pln">jack</span><span class="pun">-</span><span class="pln">admin kill</span><span class="pun">-</span><span class="pln">server
</span><span class="pun">./</span><span class="pln">prebuilts</span><span class="pun">/</span><span class="pln">sdk</span><span class="pun">/</span><span class="pln">tools</span><span class="pun">/</span><span class="pln">jack</span><span class="pun">-</span><span class="pln">admin start</span><span class="pun">-</span><span class="pln">server</span></code></pre>
</li>
</ol>
</li>
 	<li>system image too large error: Try a swap file to increase memory space or change the settings in a make file to pre-install less Google service packages</li>
</ol>
</li>
</ol>
