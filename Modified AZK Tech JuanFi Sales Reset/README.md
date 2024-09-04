# Modified version of JuanFi Sales Reset by AZK Networks
---

## 1.) Delete the existing scheduler for monthlyincome and todayincome

## 2.) Copy and paste this script on the terminal

~~~bash
/system scheduler add interval=1d name="Reset Sales" on-event=":local day [:pick [/system clock get date] 8 \
    10];:log warning \"===========<[[ Resetting Daily Sales! ]]>===========\";/system scri\
    pt set source=\"0\" todayincome;:if ([/system clock get date] ~ \"01\") do={:log warni\
    ng \"===========<[[ Resetting Monthly Sales! ]]>===========\";/system script set month\
    lyincome source=\"0\"}" policy=\
    ftp,reboot,read,write,policy,test,password,sniff,sensitive,romon start-date=\
    jan/01/2024 start-time=00:00:00
~~~
