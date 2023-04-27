---
title: Setup YCSB & MapKeeper to benchmark LevelDB
date: 2017-03-29 16:33
author: selcuk
image: 
  thumbnail: /assets/images/yahoo.png
---
LevelDB gets its own benchmark binary called "db_bench" under the directory "out-shared" after make. But there are other tools could benchmark LevelDB like MapKeeper. YCSB is a Yahoo! cloud service benchmark without the support for LevelDB. It is a possibility to constitute YCSB into MapKeeper to benchmark LevelDB. Here are my so far steps on doing so with an RHEL 7 on AWS.
<ol>
 	<li>Install Java &amp; Maven
<ol>
 	<li>
<pre><code>sudo yum install java-1.7.0-openjdk-devel
 wget http://www-eu.apache.org/dist/maven/maven-3/3.3.9/binaries/apache-maven-3.3.9-bin.tar.gz
 sudo tar xzf apache-maven-3.3.9-bin.tar.gz &amp; cd /opt
 sudo ln -s apache-maven-3.3.9 maven
 sudo vi /etc/profile.d/maven.sh -- to setup environment variable
 export M2_HOME=/opt/maven
 export PATH=${M2_HOME}/bin:${PATH}</code></pre>
</li>
</ol>
</li>
 	<li>source /etc/profile.d/maven.sh</li>
 	<li>mvn -version -- to make sure maven is installed with the right version</li>
</ol>
<ul>
 	<li>Install Libevent
<ol>
 	<li>
<pre><code>wget http://monkey.org/~provos/libevent-2.0.12-stable.tar.gz
tar xfvz libevent-2.0.12-stable.tar.gz
cd libevent-2.0.12-stable
./configure --prefix=/usr/local
make &amp;&amp; sudo make install</code></pre>
</li>
</ol>
</li>
 	<li>Install Boost
<ol>
 	<li>Boost 1.48.0
<ol>
 	<li>
<pre><code>wget http://superb-sea2.dl.sourceforge.net/project/boost/boost/1.48.0/boost_1_48_0.tar.gz
tar xfvz boost_1_48_0.tar.gz
cd boost_1_48_0
./bootstrap.sh --prefix=/usr/local
sudo ./b2 install </code></pre>
</li>
</ol>
</li>
 	<li>Boost 1.53
<ol>
 	<li>
<div class="post-text">
<pre class="lang-c prettyprint prettyprinted"><code><span class="pln">sudo yum install boost</span><span class="pun">-</span><span class="pln">devel</span></code></pre>
</div></li>
</ol>
</li>
</ol>
</li>
 	<li>Install Thrift
<ol>
 	<li>Steps
<ol>
 	<li>
<pre><code>wget http://archive.apache.org/dist/thrift/0.8.0/thrift-0.8.0.tar.gz
tar xfvz thrift-0.8.0.tar.gz
cd thrift-0.8.0
./configure --prefix=/usr/local</code></pre>
</li>
</ol>
</li>
 	<li>Make sure that
<ol>
 	<li>
<pre><code>Building C++ Library ......... : yes
Building TNonblockingServer .. : yes</code></pre>
</li>
 	<li>
<pre><code>make &amp;&amp; sudo make install</code></pre>
Bug report:
<ol>
 	<li>Include patch: https://issues.apache.org/jira/browse/THRIFT-2367</li>
 	<li>
<pre>Modified /usr/local/include/thrift/protocol/TProtocol.h and
/usr/local/include/thrift/TApplicationException.h to include &lt;stdint.h&gt;</pre>
</li>
 	<li>Include patch: https://svn.boost.org/trac/boost/attachment/ticket/6165/libstdcpp3.hpp.patch</li>
</ol>
</li>
</ol>
</li>
</ol>
</li>
 	<li>Install YCSB in MapKeeper
<ol>
 	<li>
<pre><code>cd $MKROOT/ycsb
git clone git://github.com/brianfrankcooper/YCSB.git
cd YCSB
mvn clean package</code></pre>
</li>
 	<li>
<pre><code>./bin/ycsb load mapkeeper -P ./workloads/workloada -- to see if it work</code></pre>
</li>
</ol>
</li>
 	<li>Install LevelDB on CentOS 7/RHEL 7
p.s. Libs work a little different on CentOS than it on Ubuntu or macOS
<pre><code>git clone https://github.com/google/leveldb.git
 cd leveldb &amp;&amp; make
 sudo cp include/leveldb /usr/local/include -rf
 sudo cp out-shared/libleveldb.* /usr/local/lib
 * sudo vi /etc/ld.so.conf.d/usrlocallib.conf -- let bash know you lib are here by adding the following lines
 /usr/local/lib
 sudo /sbin/ldconfig –v -- flush the settings
 ./db_bench to see if LevelDB works</code></pre>
</li>
 	<li>Run the benchmark in MapKeeper
<ol>
 	<li>cd mapkeeper/leveldb &amp;&amp; make
<ol>
 	<li>possible bug 1: libstdcpp3.hpp
<ol>
 	<li>solution: add this patch into the .hpp file: https://svn.boost.org/trac/boost/attachment/ticket/6165/libstdcpp3.hpp.patch</li>
</ol>
</li>
 	<li>possible bug 2: TIME_UTC is not correct
<ol>
 	<li>solution: change all TIME_UTC into TIME_UTC_ in that .xtime file</li>
</ol>
</li>
 	<li>possible bug 3: lboost-thread not found
<ol>
 	<li>solution: install a boost with version higher than 1.48.0</li>
</ol>
</li>
</ol>
</li>
 	<li>./bin/ycsb load mapkeeper -P ./workloads/workloada
<ol>
 	<li>I haven't done it yet without bugs so hope anyone could help? :)</li>
</ol>
</li>
</ol>
</li>
</ul>
Good Enough? Maybe
