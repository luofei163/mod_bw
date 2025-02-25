Apache2 - Mod_bw v0.92

Author       : Ivan Barrera A. (Bruce)

HomePage     : Http://Ivn.cl

Release Date : 20-Jul-2010

Status       : Stable

License      : Licensed under the Apache Software License v2.0
               It must be included as LICENSE in this package.

Platform     : Linux/x86         (Tested with Fedora Core 4, Suse, etc)
               Linux/x86_64      (Redhat Enterprise 4)
               FreeBSD/x86       (Tested on 5.2)
               MacOS X/Ppc x86   (Tested on both platforms)
               Solaris 8/sparc   (Some notes on compile)
               Microsoft Windows (Win XP, Win2003, Win7. Others should work)
               HP-UX 11          (Thanks to Soumendu Bhattacharya for testing)

Notes        : This is a stable version of mod_bw. It should work with
               almost any MPM (tested with WinNT/prefork/Worker MPM).

               We are reaching the End of mod_bw series 0.x. As soon as this
               last changes are confirmed by the users (perhaps some changes
               at request), i'll set this release to version 1.0 final.

Limitations  : This mod doesn't know how fast the client is
               downloading a file, so it just divides the bw assigned
               between the users.
               MaxConnections works only for the given scope. (i.e , all
               will limit maxconnections from all,not per ip or user)

Changelog :

2010-07-20 : Fixed ap_get_server_banner unknown on older apache version

2010-05-27 : Fixed weird behaviour on Windows Hosts. (mod_bw.txt)
             Added high resolution timers for windows. (speed improvements)
             Fixed stupid bug that caused crash when mod is enabled but there is
            not a single limit.
            
2010-05-24 : Code Cleanup. No more warnings or stuff in Visual Studio

2010-04-28 : Bruce's Birthday Gift : A callback to the stats of the mod :)

2010-04-06 : Fixed "Invisible" memory leak. Only seen when serving HUGE streams.

2010-03-10 : Fixed MinBandwidth screwing limits. Another unsigned/signed screw up.

------------------------------------------------------------------------------

  Some time ago, Jacky Yuen contacted me about a bug in the mod. It didn't look like
the mod's fault, but i took a deeper look.

The issue :

  On his windows server (win2003, and XP) the mod was working, but was unable to
reach high speeds (1MB/s). (i heard something like this before, and all you need is 
to change the packet size).
  However, doing this, didn't fix the issue. I tried this on my server, and i was
able to get about 500KB/s. Setting the packet size to 16384 gave me 1MB/s.
  Jacky told me he still had problems, so i tried all windows systems i got around.
  A windows 2003 server, a Windows 7 (32bits) showed the problem (max speed was 490KB/s)
. A Windows 7 64bits
and Win 2003 SP2 didnt.
  The most awesome part, is that if you run Google Chrome, a Flash application, or
Windows Media Player, the mod is able to deliver up to 1MB/s.

  Took me a while, but i found that Windows doesnt run a high resolution timer
all the time. In the servers with the issue, the timer run every 10 ms.
Whenever you run some of the mentioned apps, they set the timer to the highest
speed possible (in my equipment, 1 ms). This setting affects all other
applications, so the mod was able to sleep small times to deliver data as it
needed.
  So, i wrote 2 fixes. The first one, i didnt wanted to, but i made the mod to
set the timer to the highest resolution possible (it is written to the logs if
you want to peek). And the second, is to avoid waiting small times to send
data. Minimum time to sleep is 200ms now, and data is adjusted for this. (only
for windows). Both fixes are under defines that will compile just for windows,
so linux users won't notice any changes.
  The good thing, for windows users, is that with this fixes, i've been able
to get speeds up to 2.45MB/s under windows!. Using apache by itself, i just
got 1.2MB/s. Just enabling the mod and setting a high limit, the speed got
pumped up.

  That's it. Now i'll be back working on the next release.

-- (previous readme.txt text follows dated 24/May/2010)

  Again... It has been a while since i've upted the code. (work, personal
life, money issues, the same stuff we all fight daily)
  However, that doesn't mean i forgot about it. I've just been working my  
*** off. I've got many emails with suggestions, some bugs, etc..
  I'm doing one of the last updates of this line of mod_bw (0.x). It's 
mainly a couple of bugfix, and a little callback to get stats on the running
mod. I hope it wont break anything.. (of course i've tested it a lot).

  Ah!, i said this is one of the last updates. Yes. This is because this line
of code is limiting the possibilities of the mod, so i've started a new mod_bw
using other set of techniques. (so, it is highly experimental, and uses 
completely new instructions). This new branch of the mod was born thanks to
the email sent by Borislav Borislavov (icn.bg), who needed some special features
to be implemented, not possible with the current code.
  So, if you wanted the mod to do per-ip limiting, traffic limiting, etc.. 
keep checking my site. I'll be releasing a new branch of mod_bw to public soon.
(Unfortunately, it won't be released under the Apache license yet).

  Now, back to this mod :

  First, i've fixed two annoying bugs. 
  - MinBandWidth -1 was screwing things up sometimes. All because i was using 
an unsigned integer to store the -1. Yeah, my bad. 
  - The limiting handler was leaking memory. A few bytes at a time, but for
large files (really, really large) this could mean one of the apache childs
consuming almost all memory. Thanks to Christian Spielberger who insisted
and helped me to find this. (i was sure there was something.. but my servers
recycle apache childs pretty often). Fun Fact : I found the bug, was cleaning 
my test code, and he sent me an email with the exact same solution i came up.
  Great minds think alike, eh ? :) 

  Then, i made a status callback for the mod. This callback will show some 
stats on the running mod, for each memory segment used to limit bandwidth.
  Is it a simple stat. However, i received many emails asking for this.

  This is easy to achieve :

  In your admin vhost (if you have one.. If not, any vhost you want to check), 
use a location to set a handler for the callback. 

  Suppose the vhost for 127.0.0.1 :

  <location /modbw>
    SetHandler modbw-handler
  </location>


  Now you can get information of the mod by visiting http://127.0.0.1/modbw
  You can get the same information in csv format at http://127.0.0.1/modbw?csv


  Please, test this changes, let me know how it works.
  If you have some ideas (i.e. information to add in the stats), post them. If
it can or can't be done in this branch of the mod, i'll let you know. 


------------------------------------------------------------------------------


Contents :

1 .- Notices

1.1 - ChangeLog and TODO

1.2 - Important Caveats

2 .- Installing

2.1 - Windows

2.2 - Linux

3 .- Getting it to Work, Directives

3.1 - BandWidthModule

3.2 - ForceBandWidthModule

3.3 - BandWidth

3.4 - MinBandWidth

3.5 - LargeFileLimit

3.6 - BandWidthPacket

3.7 - BandWidthError

3.8 - MaxConnection 

3.9 - Status Callback

4 .- Examples

4.1 - Misc Examples

5 .- FAQ

------------------------------------------------------------------------------

1.- Notices

 1.1 - ChangeLog & TODO :

   This has been moved to the files ChangeLog and TODO. 

 1.2 - Important Caveats :

   There is a couple situations in which you might get unexpected results with
 mod_bw. From the emails i've got, most of this situations are when using
 mod_proxy, and sometimes mod_php.
   If you are using one of this mods, and you see mod_bw output in the error log
 but there is no limiting being done, the first think you have to try, is to
 change the LoadModule directive on httpd.conf. Put mod_bw directive as the
 first one in the list, and then all the others. (or at least, put it before 
 mod_proxy and mod_php). This should fix the issue.

   Also, there are some static values defined in the code, that *might* cause 
 weird issues in specific cases.
   The main two, are listed here :

    #define MAX_VHOSTS 1024
    #define MAX_BUF 1024

   For the new status page, the mod assumes you wont have more than 1024
  mod_bw enabled virtual hosts. If you do, please, change this number. It
  won't cause any troubles, it just mean the max number of "spaces" to reserve
  when storing the vhost names in memory. If you dont fix this number, only
  the first MAX_VHOSTS vhosts will show at the status page.

   The second line, is the length of the temporal space the mod uses to
  create some strings. If you have really long vhost names, you might find the
  status page shows these names truncated at some point, and show incorrect
  info. You will need to increase this number.


------------------------------------------------------------------------------

2.- How To Install :

 2.1 - Windows

   In Windows, you have to download the binary dll from the site (or compile
  one yourself) that matches your apache version, and Install it under modules
  on your apache tree. Then edit httpd.conf, and add the LoadModule sentence.
  If there is no matching binary dll with your version of apache (the latest)
  you can ask me to compile it for you. Post a message in the home site as a
  feature request.

  Example :

    Download mod_bw-2.2.14.dll from the home site, to c:\apache2\modules
    Edit httpd.conf, and add LoadModule bw_module modules/mod_bw-2.2.14.dll
    Restart Apache.

 2.2 - Linux

   Well.. (under *nix) you *have* to compile it.. (unless your distribution
  has it available as a package. Many already do ! Thanks !)

  REQUIREMENTS :

   You will need apache2 headers in the include path. In redhat/suse or others
  this mean apache-devel or httpd-devel installed. (or any other that provides
  http/apr headers and apxs2 tool)
   You also need support for SHM in your OS. Usually you will have this.

   For this to work, you MUST have apxs (or apxs2) installed. You might need
  to use its full path (i.e. /usr/sbin/apxs .....). Well, you have to do this :

   apxs -i -a -c mod_bw.c
   or
   apxs2 -i -a -c mod_bw.c

   This will bring out some stuff... nothing to worry (unless you see an error).
  If it says ok, then you are ready. Restart apache (service httpd restart on
  redhat, mandrake, and others... apachectl restart .. rcapache2 restart or... 
  well, you should know)

  Note On Solaris Users :

   If you got problems compiling it on Solaris, remember to check you have 
  libgcc correctly installed, and the ld_library_path set up correctly. 
   I didnt, so i compiled this way :

   export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/usr/local/lib
   apxs -i -a -c mod_bw.c -L/usr/local/lib -lgcc_s
               
   And then started apache, and everything worked fine.

  Note On SuSe Users :

   You NEED to have installed apache2 header files (apache2-devel rpm).
  Otherwise you'll get lots of include files error.

  If you didn't manage to get it installed, drop me a letter. But please
 be sure you tried everything. Sometimes it is just a forgotten step.
 
------------------------------------------------------------------------------

3.- Getting it to Work :


  Ok.. this is the most confusing part.
  This mod, is able to limit bandwidth usage on every virtual host or
 directory. htaccess files will not be supported on this branch. (0.x)

 * Configuration Directives
 *****************************************************************************

 3.1 - BandWidthModule [On|Off]

    You need to set this to On for the mod to work.. By default, the mod is
   disabled, and wont limit anything.

   Example :
   
              BandWidthModule On

 3.2 - ForceBandWidthModule [On|Off]
 
    By default, the mod wont catch every request.
    If you enable it, every request will be processed by the mod.
   
   Example :
   
              (normal use)
              AddOutputFilterByType MOD_BW text/html text/plain

              (enabling Force)
              ForceBandWidthModule On
 

 3.3 - BandWidth [From] [bytes/s]

    This takes 2 parameters. From is the origin of the connections. It could
   be a full host, part of a domain, an ip address, a network mask (i.e
   192.168.0.0/24 or 192.168.0.0/255.255.255.0) or all.
    The second parameter indicates the total speed available to the Origin.
    If speed is 0, there is no limit.

   Example :
   
            BandWidth localhost 10240
            BandWidth 192.168.218.5 0

            ( Order is relevant. First entries have precedence )


    As for version 0.8, an user agent matching capability was introduced.
    If you want to limit all clients using certain browser, you can limit
    doing this :

            BandWidth u:[User-Agent] [bytes/s]

    User agent is a regular expression which will match the one sent by the
    browser. This is easier to explain with examples :

   Example :
   
            BandWidth "u:^Mozilla/5(.*)" 10240
            BandWidth "u:wget" 102400
            
    First one, will match a browser that identifies itself as Mozilla/5(etc)
   Second one, will match a browser that has wget in any part of its id.


 3.4 - MinBandWidth [From] [bytes/s]

    This takes 2 parameters. From is the origin of the connections. It could
   be a full host, part of a domain, an ip address, a network mask (i.e
   192.168.0.0/24 or 192.168.0.0/255.255.255.0) or all.
    The second parameter indicates the minimun speed each client will have.
   What does this mean ? If you have a total of 100kbytes speed, and you put
   MinBandWidth at 50kbytes, it doesnt matter how many clients you have, all
   of them will have a minimun of 50kbytes of total speed to download.
    If speed is 0, you will be using the default minimun (256 bytes/s).
    There is a special value of -1. This value means that each client will
   have a top speed determined by the BandWidth directive. See the examples.

   Examples :
   
              BandWidth    all 102400
              MinBandWidth all 50000
 
    The example above will set a top speed of 100kb for the 1º
   client. If more clients come, it will be splitted accordingly but
   everyone will have at least 50kb (even if you have 50 clients)

              BandWidth    all 50000
              MinBandWidth all -1

    This example, makes everyone have 50kb as top speed.

 3.5 - LargeFileLimit [Type] [Minimum Size] [bytes/s]

    Type, is the last part of a file, or * for all. You can use .tgz to match 
   only tar-compressed files, .avi to match video files, or * to match all.
    Minimum Size, is the size (in kbytes) of the file, to be matched. That way
   you can match huge video files that hog your bandwidth.
    The last parameter... is obvious. The speed allowed.

   Example :
   
             LargeFileLimit .avi 500 10240

    This limits .avi files over (or equal to) 500kb to 10kbytes/s

 3.6 - BandWidthPacket [Size]

    Probably you never need to touch this. It defaults to 8192 which is good
   for almost any speed.
    It must be a size between 1024 and 131072. A Small packet will cause the
   top speed to be lower, and the mod using more time splitting. If you use
   a Size too big, the mod will adjust it to lower speeds.
    If you are using the mod in high speed networks, this is, you want to
   set limits of megabits/s, you will be better using packet sizes of
   16384, or 32768. 

 3.7 - BandWidthError [Error]

    This directive is useful to deliver a personalized error code.
    By default, when maxconnections is reached, the mod will issue a 503 
   HTTP_SERVICE_UNAVAILABLE code. For some users, it is annoying to have an
   error message, and don't knowing why. You could use an ErrorDocument to 
   point error 503 to a page explaining that you are under a heavy load of
   connections, but sometimes 503 is issued by the server for other reasons.
    So, with this directive, you can set the error code to return when
   maxconnections is reached. You can use any error code between 300 and 599.
   Please note, that some of the error codes are already used, so before using
   any number, take a look to a list of the codes (search for http error codes
   in google). 
    When testing, i've used the error code 510, which hasn't been defined yet.

    And Example, with Personalized Error Page :

      ErrorDocument 510 /errors/maxconexceeded.html
      BandWidthError 510

    Note : Sometimes, the personalized page didn't appear. I'm not sure, but
    in many cases, it got fixed, by making the page size over 1024bytes.
    Anyways, if you need help using ErrorDocument, refer to the apache
    Documentation.
 
 3.8 - MaxConnection [From] [Max]

    This takes 2 parameters. From is the origin of the connections. It could
   be a full host, part of a domain, an ip address, a network mask (i.e
   192.168.0.0/24 or 192.168.0.0/255.255.255.0) or all.
    The second parameter, is the max connections allowed from the origin. Any
   connection over Max, will get a 503 Service Temporarily Unavailable

    There is a catch. You NEED to have a BandWidth limit for the same origin.
   It doesnt need to be a low limit. But you need one. (unlimited, doesn't
   count)
    You might wonder why. It's because im using them same memory space of the
   bandwidth limit to count the connections, so i can save memory space.
    If you dont put a BandWidth using the same origin, MaxConnections will be
   ignored.

   Example :

              BandWidth all 102400000
              MaxConnection all 20
              
    or
    
              BandWidth all 102400000
              BandWidth 192.168.0.0/24 10240
              MaxConnection all 20
              MaxConnection 192.168.0.0/24 5


    As for version 0.8, an user agent matching capability was introduced.
    If you want to limit all clients using certain browser, you can limit
    doing this :

            MaxConnection u:[User-Agent] [Max]

    User agent is a regular expression which will match the one sent by the
    browser. This is easier to explain with examples :

   Example :
   
            MaxConnection "u:^Mozilla/5(.*)" 5
            MaxConnection "u:wget" 5

   First one, will match a browser that identifies itself as Mozilla/5(etc)
  Second one, will match a browser that has wget in any part of its id.

  Please, rememeber that every speed, will depend mostly on your connection.
 You can't get more speed if you dont have it.

  Remember also.. if you dont follow the instructions, and get some weird 
 results, recheck your config before sending me an email.

  3.9 - Status Callback
  
    Since v0.9, the mod can display a simple status page, showing stats from
  the running mod. This stats show the exact information being used by the mod
  to do the limiting in that second.

    For this to work, you need to set a handler on any vhost. You might want 
  to set this under an admin vhost, or set some policies to make it private.
  Your call.

    Example (let's assume the vhost is for 127.0.0.1) :
  
  <location /modbw>
    SetHandler modbw-handler
  </location>


    Now, you can get the status info at http://127.0.0.1/modbw
    ( Or download a CSV of the stats at http://127.0.0.1/modbw?csv )

    The information provided is the following :

    id   : 0                // This is just a correlative number for each config.
    name : work.ivn.cl,all  // The vhost name, and the scope of the rule
    lock : 0                // If the memory segment is being used (0 = no)
    count: 0                // Number of users connected to this rule
    bw   : 0                // Bandwidth currently being used by the rule
    bytes: 0                // Number of bytes last sent. Only true if count>0
    hits : 0                // Number of times anyone has accesed this rule.

    Simple, yet useful !

------------------------------------------------------------------------------

4.- Examples

  4.1 - Misc examples

    Limit every user to a max of 10Kb/s on a vhost :

    <Virtualhost *>
      BandwidthModule On
      ForceBandWidthModule On
      Bandwidth all 10240
      MinBandwidth all -1
      Servername www.example.com
    </Virtualhost>


    Limit al internal users (lan) to 1000 kb/s with a minimum of 50kb/s , and 
  files greater than 500kb to 50kb/s.

    <Virtualhost *>
      BandwidthModule On
      ForceBandWidthModule On
      Bandwidth all 1024000
      MinBandwidth all 50000
      LargeFileLimit * 500 50000
      Servername www.example.com
    </Virtualhost>


    Limit avi and mpg extensions to 20kb/s.

    <Virtualhost *>
      BandwidthModule On
      ForceBandWidthModule On
      LargeFileLimit .avi 1 20000
      LargeFileLimit .mpg 1 20000
      Servername www.example.com
    </Virtualhost>


    Using it the "right" way, with output filter by mime type (for text) 
  to 5kb/s:
    
    <Virtualhost *>
      BandwidthModule On
      AddOutputFilterByType MOD_BW text/html text/plain
      Bandwidth all 5000
      Servername www.example.com 
    </Virtualhost>


    If you need help on doing more complex setup, post over GitHub.

------------------------------------------------------------------------------

5.- FAQ

(No particular order)

1.- Why should i use mod_bw ?

   If you want to restrict the top speed a site is able to use, or limit the
  max connections allowed per site, or just to try the mod.
    Some people told me, they use it primarily to stop small sites hogging 
  all the bandwidth when serving video, images, or other content.

2.- How do i ... ?

   First, read the documentation. it is pretty straightforward to use.
  If you can't make it work, or if you want to ask for a feature, visit the
  GitHub site, and post a request. Remember to read the documentation and the
  faq. If the request is already posted, i'll just delete the duplicates.

3.- What's the difference with mod_bwshare, mod_throttle, etc ?

   The main difference, is that this mod, is aimed to the Apache 2 API.
  Some other differences, is, how this mod works, and the directives it 
  implements.
    I took some ideas from other mods, as the shared memory implementation
  (mod_bandwidth2 from Tim Verhoeven). Most of the directives, are in origin
  from mod_bandwidth for apache 1.

4.- How does it works ?

   The mod, will set a shared memory holding all of the configuration you
  make. In this space, it will also, keep a "count" of the info currently
  using (as current connections, bw used, time, and bytes sent).
    When you assign a bw limit, the mod will "split" the data, and will send
  it piece by piece, with a small delay between pieces. The delay will be
  adjusted so at least 1 piece is sent in a second, thus eviting browser
  timeout.
    If there are two or more clients downloading from the same vhost, the
  limit will be "splitted" too. So, if you have a bw limit of 16Kb and two
  clients, each one will have 8Kb to download. You can also set a fixed 
  download rate, so each client will have a fixed maximun download rate.
    This is a simple explanation of how this works. Take a look at the code
  if you want to know more about it.

5.- Can you make it do ... ? 

   Post a request for a feature through GitHub.

6.- Can you make mod_bw work for ... ?

   See question 5. Anyways, if i dont own (or have access) to the hardware
  needed, i won't be able to help you.

7.- I'm having some trouble in windows ... 

   Well, thanks to the guys at Microsoft, who kindly offered access to MSDN
  to the Apache Community, the dll's i post on my site are produced using
  licensed Visual C, and i can help with any Windows Issue!
    Post or send me your problem. I'll be glad to (try to) help. 

8.- The mod is not limiting certain Directory

   Ok... first, read the documentation. Then the FAQ. If it isn't working,
  then look if you have defined correctly the limits within that directory.
    If you have a limit in a vhost, and inside the vhost, a directory with
  another limit, a new context with the configs of the directory is created,
  and only have the configurations you added there. 

  In example :

 <VirtualHost *>
    BandWidthModule On
    BandWidth all 16384
    LargeFileLimit * 500 4096
    <Directory />
      LargeFileLimit * 100 1024
    </Directory>
 </VirtualHost>

  This wont limit Directory / to 16384. The Directory wont "inherit" the
 settings from the vhost if you use some of the mod's directives.

------------------------------------------------------------------------------

 Ivan Barrera A. 
 
 Ivn Systems Software 

# add by luofei
centos7.6 
yum install httpd-devel-2.4.6-93.el7.ns7.01.x86_64
/bin/apxs   -i -a -c mod_bw.c


 
 
