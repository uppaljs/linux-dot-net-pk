---
title: Opendiameter Compilation Instructions
author: uppal
type: post
date: 2008-09-05T00:29:17+00:00
url: /opendiameter-compilation-instructions/
categories:
  - Howtos
tags:
  - AAA
  - charging
  - diameter
  - opendiameter

---
**For FC8**

1. Make sure gcc , gcc-c++ , bison , flex , make , openssl and openssl-devel are present

2. yum -y install boost-devel

3. export BOOST_ROOT=/usr/include/

4. Get ACE [Here][1] .

5. tar xvf ACE-5.5.tar.gz

6. cd ACE_wrappers

7. mkdir build

8. cd build

9. Patch the configure file

&#8212; configure.orig 2008-08-05 11:11:19.000000000 -0800  
+++ configure 2008-08-05 11:13:30.000000000 -0800  
&#8230;  
@@ -10329,7 +10329,7 @@ _ACEOF  
*)  
{ echo &#8220;$as_me:$LINENO: enabling GNU G++ visibility attribute support&#8221; >&5  
echo &#8220;$as_me: enabling GNU G++ visibility attribute support&#8221; >&6;}  
&#8211; ACE\_GXX\_VISIBILITY_FLAGS=&#8221;-fvisibility=hidden -fvisibility-inlines-hidden&#8221;  
+ ACE\_GXX\_VISIBILITY_FLAGS=&#8221;-fvisibility=hidden&#8221;  
ACE\_CXXFLAGS=&#8221;$ACE\_CXXFLAGS $ACE\_GXX\_VISIBILITY_FLAGS&#8221;  
cat >>confdefs.h <<_ACEOF

10. cd build ; ../configure

11. make && make install

12. export ACE\_ROOT=/usr/local/src/ACE\_wrappers/

13. tar xvf opendiameter-1.0.7-i.tar.gz

14. cd opendiameter-1.0.7-i

15. ./configure

16. make

17. make install

<!-- AdSense Now! Lite: PreFiltered - NoAds [ WP is not in the loop. ] -->

 [1]: http://download.dre.vanderbilt.edu/previous_versions/ACE-5.5.tar.gz