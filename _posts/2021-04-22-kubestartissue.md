---
layout: post
title: MFdca - kube-start.sh showing time out error
author: thu4nvd
subtitle: in life of support MFSA
date: 2021-04-22 05:10:00 +0800
categories: [IT, Automation]
tags: [mfdca, automation]
---

## Issue

Product: [Data Center Automation](https://docs.microfocus.com/itom/Data_Center_Automation:2019.11/Home)  
Version: 2019.11  
At time of executing ./kube-satrt.sh it's showing time-out error at docker service.
```
./kube-start.sh: line 93: ((: n > /usr/bin/timeout -s9 5 : syntax error: operand expected (error token is "/usr/bin/timeout -s9 5 ")))
```

![kubestart](/assets/img/kubestart.PNG)
_Error when run kube-start.sh_

## Analysis

After commpare with another script: kube-start.sh on the lab environment, there is diffirent at line 93.

(Use Notepad++ or diff command to compare the difference)

Ask customer to verify:
```
I suppose there is syntax error of the script kube-start.sh
May I know if did the script work previously?
                Is there any change in the environment or on the script? 
                (update, copy paste the scripts, etc…)
                Change of Shell execution : output of #echo $SHELL
```


## Root cause

There is swap memory issue in DCA and because of it Kubectl command is not working. 
So, at that time customer have stop and restart the Kubernetes services. 
He has tried multiple times this command “./kube-start.sh”  but it showing error.

## Solution

FYI- Swap memory issue is resolve now
There is no change in DCA environment or in the script

But today again I have checked the restart services its working fine maybe because of swap issue it’s not working at that time.
