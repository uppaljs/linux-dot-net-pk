---
title: 'ServiceXpress Framework :: ChrisXpress Browserless Solution'
author: uppal
type: post
date: 2010-08-19T16:19:33+00:00
url: /servicexpress-framework-chrisxpress-browserless-solution/
categories:
  - Howtos

---
Hi,

Had a nightmare of a task yesterday to configure SX for faster access over VPN. The only solution was to do it browser less , anyone who knows and / or has used ChrissXpress will agree , that its extremely slow and tightly bound around Java 1.3.1 .

The solution was to download all of the jar files from the framework and copy them over to JAVA_HOME/lib and create a small script to launch it by clicking.

Following is the script i slapped together for achieving this task , it automatically setups the $CLASSPATH & $PROPERTIES FILE which are required for SX to run. The properties file is project specific and needs to be collected from the platform it self.

> <div id="_mcePaste">
>   set CLASSPATH=
> </div>
> 
> <div id="_mcePaste">
>   for %%i in (*.jar) do call addpath.bat %%i
> </div>
> 
> <div id="_mcePaste">
>   echo CLASSPATH : %CLASSPATH%
> </div>
> 
> <div id="_mcePaste">
>   echo CLASSPATH : %CLASSPATH% >> SXClientLog.txt
> </div>
> 
> <div id="_mcePaste">
>   rem   &#8212;&#8212;&#8212;&#8212;&#8212;&#8212;&#8212;&#8212;&#8212;&#8212;&#8212;&#8212;
> </div>
> 
> <div id="_mcePaste">
>   rem   Call java
> </div>
> 
> <div id="_mcePaste">
>   rem   &#8212;&#8212;&#8212;&#8212;&#8212;&#8212;&#8212;&#8212;&#8212;&#8212;&#8212;&#8212;
> </div>
> 
> <div id="_mcePaste">
>   echo Starting&#8230;
> </div>
> 
> <div>
>   <span style="font-family:Arial, Helvetica, sans-serif;font-size:12px;color:#666666;"></p> 
>   
>   <div id="_mcePaste">
>     java -classpath “%CLASSPATH%” -Xmx160m at.siemens.servicexpress.sxfw.capp.appbase.SxJApplet %PROPERTY_FILE% >> SXClientLog.txt 2>&1
>   </div>
>   
>   <p>
>     </span></div> </blockquote> 
>     
>     <div>
>       Following is the code for addpath.bat
>     </div>
>     
>     <blockquote>
>       <div>
>         <div>
>           echo Append CLASSPATH with %1 >> SXClientLog.txt
>         </div>
>         
>         <div>
>           set CLASSPATH=%1;%CLASSPATH%
>         </div>
>         
>         <div>
>           shift
>         </div>
>         
>         <div>
>           :args
>         </div>
>         
>         <div>
>           if &#8220;%1&#8243;==&#8221;&#8221; goto noargs
>         </div>
>         
>         <div>
>           echo Append CLASSPATH with %1 >> SXClientLog.txt
>         </div>
>         
>         <div>
>           set CLASSPATH=%1;%CLASSPATH%
>         </div>
>         
>         <div>
>           shift
>         </div>
>         
>         <div>
>           goto args
>         </div>
>         
>         <div>
>           :noargs
>         </div>
>       </div>
>     </blockquote>
>     
>     <div>
>
>     </div>
>     
>     <div>
>       That&#8217;s it , just start by running the batch file on the top and you&#8217;ll have a working offline CX installation.
>     </div>
>     
>     <div>
>       🙂
>     </div>
>     
>     <div>
>
>     </div>
>     
>     <!-- AdSense Now! Lite: PreFiltered - NoAds [ WP is not in the loop. ] -->