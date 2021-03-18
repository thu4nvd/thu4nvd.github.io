---
layout: post
title: How do I be a good DevOps engineer?
subtitle: Struggling question from "Mukashi"
gh-badge: [star, fork, follow]
categories: [Blogging, howto]
tags: [devops, howto]
comments: true
---

# Definitions

The answer to the 1st question is easy to write and pretty hard to implement. You can be a "Good" Devops Engineer by being good to your fellow Devs, Ops and QAs. In a project, there will be many scenarios where your fellow Devs and QAs and Devops(you) will do things wrong. Its hard to "KEEP CALM AND CARRY ON" when there are blunders happening in the project. Its even more difficult to keep calm when some action by someone hampers your work. So how do you handle this?

Learn how that job is done. You may not necessarily be perfect, but you at least know how its been done. That way you get to know the obstacles they may have faced while working. I will give you 3 examples of how "Good Engineers" handled a situation.

#### Dev Mistake

A developer wrote a code that was written on AppleOS and deployed on Ubuntu for QA to test with the developer on leave thereafter and release due in the next few days. Needless to say the code was well tested. Somehow it did not work well. After researching a little the QA found that the header file mentioned in the code was filesutils.h while Ubuntu identified it as "FileUtils.h. Otherwise the code was perfect. The QA quickly made the changes, tested and sent a pull request with the change and updated the dev as well. Saved time and was "Good" too.
    
#### QA Mistake: 

A QA person while verifying DB on production fired a Query "DELETE" query instead of a "SELECT" The result was 24 important elements got deleted from production. The QA went to the Devops and said, "I think I executed a delete on production, I do not see my data and nor the query". The ops guy says, "No issues, its an easy one to get back, nothing to panic about". Process to get back the deleted data is, (i) Copy previous days data(around 20 GB) from S3 to local (ii) create a new database and copy the dump to the db (iii) Take present db dump (iv) copy it to local (v) create another db and copy this dump to the new db (vi) check the diff (vii) insert the diff elements from old db to the production db. (There are better ways to do this) In short, a good few hours process. The Ops guy maintained his calm, got all the required info and fixed the issue without any frustration.
    
#### Ops Mistake: 

mounted a drive /dev/xvdd instead of /dev/xvdb and deleted all the contents from it. The redis data got deleted, with redis running. Contacted developers who knew a little operations to help. They asked to not stop redis and to do a bgsave (background save utility) and got all the data back.

Mistake happen. Learning from those mistakes is important. So learning and understanding things from others perspective and being polite and calm is really essential to be a Good Devops Engineer.

# Technically

To be a Devops Engineer you need the following:

- Passion to automate almost anything
- Linux Administration Skills
- Expert in one or more scripting language
- Virtualization
- Understand the Cloud
- At least one Configuration Management tool
- Monitoring tools
- Great Troubleshooting skills
- Containerization
- At least one CI/CD tool
- Patience ... :)