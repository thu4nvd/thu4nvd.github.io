---
layout: post
title: SA-OO Integration - Lost Job ID in SA Client
subtitle: A issue on SA-OO integration
thumbnail-img: /assets/img/microfocus.png
tags: [mfsa, mfoo]
comments: true
---

We checked all the possible ways to find the JobIDs but not able to find in SA Client.
Run Job from OO, then there is some flow it has Job ID shown on SA Client, but some are missing.

# Issue Description

### System Information


SA Version: 10.60  
OS details: RHEL  
MM/Single core : Multimesh  
Integration with OO and run flow from OO to trigger Job on SA.  

### Description

We checked all the possible ways to find the JobIDs but not able to find in SA Client.
Run Job from OO, then there is some flow it has Job ID shown on SA Client, but some are missing.

-	Session 19063130001 : Flow run faillure on OO Central, output the ID and did not show the Job ID on SA Client

![showID](/assets/img/0315-01.png)

-	Session 19067880001: Flow run successfully on OO Central, output the ID and show the Job ID on SA Client

![showID](/assets/img/0315-02.png)

- On SA Client

![showID](/assets/img/0315-03.png)

# Solution

After managed node make it known OS from unknown OS it’s working fine and job not got failed and job id got created.

Check OS type on SA CLient GUI, or even run bs_software command from SA Agent of both servers ( to compare).

bs_software and bs_hardware are two script files installed by the agent installer in the following location:

~~~
%ProgramFiles%\Opsware\agent\pylibs\cog #Windows
/opt/opsware/agent/pylibs/cog #Unix
~~~


