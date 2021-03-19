---
layout: post
title: MFSA - Unable to connect SA Client to SA Core
author: thu4nvd
subtitle: in life of support MFSA
date: 2021-03-17 05:10:00 +0800
categories: [IT, Automation]
tags: [mfsa, automation]
---

## Issue

Product: [Server Automation](https://docs.microfocus.com/itom/Server_Automation:2018.08/Standard_Install_Guide/InstallingAdditionalSliceComponentBundles)  
Unable to access SA Core primary from SA Client

![sa-client](/assets/img/saclient.png)
_Error when access SA Core_

## Analysis

It is happening before the NGUI even tries to talk to the twist.

At this stage in the startup, it is attempting to talk to the OCC, via the httpsProxy port 80 redirector.

Refer the architecture of SA [here](https://docs.microfocus.com/itom/Server_Automation:2018.08/SA_overview/adv_architecture/AdvancedSAArchitecture)

![sa-architecture](https://docs.microfocus.com/mediawiki/images/2/2d/10.60/Resources/Images/SA_10_ArchNetDiagrams3.png)
_SA Architecture diagram_

### Httpsproxy

Below is the corresponding log record of the failed redirection from the httpsProxy logs:

```shell
$ cd /var/log/opsware/httpsProxy
$ grep /jnlp redirector_request_log | grep ' 74' | grep 49-31

[03/Mar/2021:09:19:31 +0000] 172.23.224.69 "GET /jnlp?hostname=172.23.224.32&launcherversion=2.1.13&-Dngui.log.file=C%3A%5CUsers%5Cdxc_shivarajup%5Cappdata%5CRoaming%5CHPE+SA%5C%5Clogs%5Cngui_03-03-2021_14-49-31.log&-Dngui.home=C%3A%5CUsers%5Cdxc_shivarajup%5Cappdata%5CRoaming%5CHPE+SA%5C&-Dngui.debug=true&-Dcom.opsware.bootstrap.host=172.23.224.32 HTTP/1.1" 74
```

The return size of "74" indicates this is a failure interaction.  The return size should be larger than this.  Below is the most recent successful NGUI download attempt from the httpsProxy logs:

```shell
$ grep /jnlp redirector_request_log | grep -v ' 74' | tail -1

[26/Jul/2020:09:02:18 +0530] 172.23.251.24 "GET /jnlp?hostname=172.23.224.32&launcherversion=2.1.13&-Dngui.log.file=C%3A%5CUsers%5Cdxc_nishanta%5Cappdata%5CRoaming%5CHPE+SA%5C%5Clogs%5Cngui_26-07-2020_09-02-18.log&-Dngui.home=C%3A%5CUsers%5Cdxc_nishanta%5Cappdata%5CRoaming%5CHPE+SA%5C&-Dcom.opsware.bootstrap.host=172.23.224.32 HTTP/1.1" 3161
```

Below are the redirector rules from the httpsProxy configuration file "/etc/opt/opsware/httpsProxy/httpd.conf":

```
Secure redirector in apache
<VirtualHost *:80>
. . .
    RewriteRule ˆ/jnlp(.*) http://localhost:9080/com.opsware.file/jnlp$1 [P,L]
. . .
</VirtualHost>
```

I've highlighted the RewriteRule that is in play for this JNLP NGUI download interaction.  
You can see that it redirects the request to port 9080, and rewrites the URL to make use of the "com.opsware.file" servlet.  
Port 9080 belongs to the OCC as you can see below:

```shell
$ lsof -Pn -i :9080 | grep LIST
java    13835  occ  405u  IPv6 125761805      0t0  TCP 127.0.0.1:9080 (LISTEN)

$ ps auxww| grep 13835
occ      13835  0.3  3.2 4709624 327464 ?      Sl   Jan21 282:00 /opt/opsware/openjdk1.8/bin/java -server -Xms384m -Xmx1024m -XX:MaxMetaspaceSize=256m -XX:NewRatio=3 -Dsun.lang.ClassLoader.allowArraySyntax=true -Docc.home=/opt/opsware/occ -Docc.cfg.dir=/etc/opt/opsware/occ -Dopsware.deploy.urls=/opt/opsware/occ/deploy/ -Djava.awt.headless=true -D[Standalone] -Dlogging.configuration=file:/opt/opsware/occ/wildfly/standalone/configuration/logging.properties -Djboss.server.name=occ -Djboss.server.temp.dir=/var/opt/opsware/occ/tmp -Djboss.server.data.dir=/var/opt/opsware/occ/data -Djboss.server.log.dir=/var/log/opsware/occ -Djboss.server.config.dir=/opt/opsware/occ/wildfly/standalone/configuration -Djboss.home.dir=/opt/opsware/wildfly13 -Djboss.server.base.dir=/opt/opsware/wildfly13/standalone -DsslProtocols=TLSv1.2 -jar /opt/opsware/wildfly13/jboss-modules.jar -mp /opt/opsware/wildfly13/modules:/opt/opsware/occ/wildfly/modules org.jboss.as.standalone --server-config=standalone-full.xml
```

### OCC component

So then the next step is to inspect the OCC logs to see what it's problem is.  
When we grep for the "com.opsware.file" deployment in the server.log from March 10th, we see the following:

```shell
$ grep com.opsware.file /var/log/opsware/occ/server.log

2021-03-10 06:49:41,629 INFO  [org.jboss.as.server.deployment.scanner] (DeploymentScanner-threads - 1) WFLYDS0004: Found com.opsware.file.ear in deployment directory. To trigger deployment create a file called com.opsware.file.ear.dodeploy
2021-03-10 06:49:42,417 INFO  [org.jboss.as.server.deployment] (MSC service thread 1-5) WFLYSRV0027: Starting deployment of "com.opsware.file.ear" (runtime-name: "com.opsware.file.ear")
2021-03-10 06:49:42,551 INFO  [org.jboss.as.server.deployment] (MSC service thread 1-8) WFLYSRV0207: Starting subdeployment (runtime-name: "com.opsware.file.war")
2021-03-10 06:49:46,934 ERROR [org.jboss.as.controller.management-operation] (Controller Boot Thread) WFLYCTL0013: Operation ("deploy") failed - address: ([("deployment" => "com.opsware.file.ear")]) - failure description: {"WFLYCTL0180: Services with missing/unavailable dependencies" => undefined}
. . .
```

What we see here is that "com.opsware.file" failed to deploy.  
When we open up the server.log and look deeper we see the reason is that the OCC is failing to authenticate to the twist using the username "buildmgr".  (The OCC interacts with the twist under the username "buildmgr".)

```shell
2021-03-10 06:49:44,914 ERROR [occ.auth.util] (ServerService Thread Pool -- 61) Error getting system user: OpswareError: opsware.twistConnectionFailed [ module: com.opsware.client.Twist, line: 0, hostname: twist, timestamp: 1615358984914 ]

com.opsware.fido.AuthenticationException: ID:   HPSA-1114
Code:   com.opsware.fido.FidoMessageSpec.AUTHENTICATION_FAILURE
Details:   Authentication failed for user buildmgr.  Details <class=UserImpl><method=authenticate><message='login incorrect for user: buildmgr'>
```

### Twist log

When we look at the corresponding twist/server.log at the same timestamp, we see a "violated signature" error:

```shell
2021-03-10 06:49:44,841 WARN Thread-312 [com.opsware.fido.impl.user.UserImpl] [authenticateInternal] Caught Exception
java.lang.RuntimeException: <ERROR> <Bean.ejbLoad> violated signature
        at com.opsware.fido.ejb.entity._AaaUser.ejbLoad(_AaaUser.java:424)
```

## Solution

So the root cause is that the user signature for "buildmgr" is wrong.  To correct this, run the following:

```shell
/opt/opsware/support/bin/verify_user_sigs buildmgr
```

In fact, I recommend running the following in order to check all SA user records:

```shell
/opt/opsware/support/bin/verify_user_sigs
```

Once all user record signatures have been fixed, then restart all of SA and try to log in again.
