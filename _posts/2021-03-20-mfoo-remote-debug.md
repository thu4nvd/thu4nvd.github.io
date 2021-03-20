---
layout: post
title: MFOO - Remote Debug :Unable to connect OO Studio to OO Central
author: thu4nvd
subtitle: in life of support MFOO
date: 2021-03-20 05:10:00 +0800
categories: [IT, Automation]
tags: [mfoo, automation]
---

## Issue

Product: [Operations Orchestraion](https://docs.microfocus.com/itom/Operations_Orchestration:10.80/Get_Started/Get_Started)  
Remote Debug :Unable to connect OO Studio to OO Central

![remote-debug](/assets/img/remote-debug1.png)
![remote2-debug](/assets/img/remote-debug2.png)
_Error when trying to remote debug from MFOO_

## Analysis

This is guide to the [Remote Debugging](https://docs.microfocus.com/itom/Operations_Orchestration:10.80/Use/Authoring_With_OO_Studio/Debugging_Central_Studio)

Issue happen when trying to connect Remote Debug
**Note that**: URL should be the URL ( not in IP)

![Remote-Debug](https://docs.microfocus.com/mediawiki/images/0/05/10.80/Resources/Images/Studio/edit_connections.PNG)
_Remote Debugger connections_

This is remarkable point, because in the Case, customer use IP instead.

### Check studio.log

Ideas is to go check _C:\Users\<user_login>\.oo\logs\studio.log_  first:

```
2021-03-16 12:44:30,349 ERROR    EnginesConnectionsServiceImpl$BasicAuthenticationCommonsHttpInvokerRequestExecutor.doExecuteRequest:542    Error invoking remote service https://15.120.137.27:8443/oo/central-remoting/engineExecutionFacade
2021-03-16 12:44:30,350 WARN     EnginesConnectionsServiceImpl.pingEngineServices:398    Could not access HTTP invoker remote service at [https://15.120.137.27:8443/oo/central-remoting/engineExecutionFacade]; nested exception is javax.net.ssl.SSLPeerUnverifiedException: Host name '15.120.137.27' does not match the certificate subject provided by the peer (CN=mc4t02388.itcs.softwaregrp.net)
```

With this error: **SSLPeerUnverifiedException** , basic ideas is to verify the client.truststore of Studio.

This command to print the certificate information:
```cmdline
cd "C:\Program Files\Micro Focus\Operations Orchestration\java\bin" 
keytool.exe -export -keystore "C:\Program Files\Micro Focus\Operations Orchestration\central\var\security\key.store" -alias tomcat -file central_public.cer
keytool.exe -printcert -v -file central_public.cer
```

If on Linux, search for keytool by command:
```shell
find / -type f -name "keytool"
```
Also can idientify the location of installation binary. 

### Document guide

Refer: [Security_Hardening_OO/Import_Certificate_to_Debugger](https://docs.microfocus.com/itom/Operations_Orchestration:10.80/Administer/Security_Hardening_OO/Import_Certificate_to_Debugger)

Export certificate from the key.store on Central and import to the Studio client.truststore.

![export](/assets/img/export-cert.png)
![import](/assets/img/import-cert.png)

## Solution

In the setting, change the URL set to FQDN instead IP address.