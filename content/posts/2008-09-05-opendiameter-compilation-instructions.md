---
title: "Opendiameter Compilation Instructions"
date: 2008-09-05
categories:
  - Howtos
tags:
  - aaa
  - charging
  - diameter
  - opendiameter
draft: false
---

**For FC8**

1. Make sure gcc, gcc-c++, bison, flex, make, openssl and openssl-devel are present

2. Install boost-devel:
```bash
yum -y install boost-devel
```

3. Set environment variable:
```bash
export BOOST_ROOT=/usr/include/
```

4. Get ACE [Here](http://download.dre.vanderbilt.edu/previous_versions/ACE-5.5.tar.gz).

5. Extract and build ACE:
```bash
tar xvf ACE-5.5.tar.gz
cd ACE_wrappers
mkdir build
cd build
```

6. Patch the configure file:

```diff
--- configure.orig 2008-08-05 11:11:19.000000000 -0800
+++ configure 2008-08-05 11:13:30.000000000 -0800
...
@@ -10329,7 +10329,7 @@ _ACEOF
*)
{ echo "$as_me:$LINENO: enabling GNU G++ visibility attribute support" >&5
echo "$as_me: enabling GNU G++ visibility attribute support" >&6;}
- ACE_GXX_VISIBILITY_FLAGS="-fvisibility=hidden -fvisibility-inlines-hidden"
+ ACE_GXX_VISIBILITY_FLAGS="-fvisibility=hidden"
ACE_CXXFLAGS="$ACE_CXXFLAGS $ACE_GXX_VISIBILITY_FLAGS"
cat >>confdefs.h <<_ACEOF
```

7. Configure and build:
```bash
cd build
../configure
make && make install
```

8. Set ACE_ROOT:
```bash
export ACE_ROOT=/usr/local/src/ACE_wrappers/
```

9. Build OpenDiameter:
```bash
tar xvf opendiameter-1.0.7-i.tar.gz
cd opendiameter-1.0.7-i
./configure
make
make install
```
