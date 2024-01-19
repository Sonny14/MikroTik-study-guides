# Login and Logout script by FJO
---
## Telegram Notification | Random Mac Sync Fix | UserTXT Based Validity format

### 1.) Paste these on login and logout script in the user profile
> - Remember to change the "telegram id" and "token" as well as the hostpot folder
#### On Login Script
~~~
### Working and Tested Scripts 01July2022
#     last modified 13 Sept 2022
#########################################################################################
# Modified based on Juanfi v4.0-v3.3 ON-Logon Script able to used with 3.2 BIN w/ 3.3 portals
# Revised By: fjoCharmedones
#########################################################################################
# DESCRIPTION: Hotspot Users ON-Login Event to generate a UserTXT Based Validity format
#########################################################################################
### enable telegram notification, change from 0 to 1 if you want to enable telegram
:local enableTelegram 1;
###replace telegram token
:local telegramToken "2021159313:AAHEBoOLogYjLCpSwVeKPVmKKO4TIxa02vQ";
###replace telegram chat id / group id
:local chatId "-XXXXXXXX";
### enable Random MAC synchronizer
:local enableRandomMacSyncFix 1;
### hotspot folder for HEX put flash/hotspot for haplite put hotspot only
:local hotspotFolder "flash/hotspot";
:local HSuser $user;
:local HSemail "bots@juanfi.local";
:local com [/ip hotspot user get [find name=$user] comment];
/ip hotspot user set comment="" $user;
 
#BOF
/do {
{
:local botmail 
:set $botmail [/ip hotspot user get [find name=$HSuser] email];

if ("$botmail"="$HSemail") do={
/system logging enable 0
:log warning "ProcessingReloadValiditySection"
/system logging disable 0;
	:local mac $"mac-address";
	:local macNoCol;
	:for i from=0 to=([:len $mac] - 1) do={ 
	  :local char [:pick $mac $i]
	  :if ($char = ":") do={
		:set $char ""
	  }
	  :set macNoCol ($macNoCol . $char)
	}
	/file print file="$hotspotFolder/data/$macNoCol" where name="noname.txt"; 
	:delay 3s; 
	/file set [find name="$hotspotFolder/data/$macNoCol.txt"] contents="";
	:delay 1s;	
 :local validUntil [/sys scheduler get $HSuser next-run];
 /file set "$hotspotFolder/data/$macNoCol.txt" contents="$HSuser#$validUntil";
 	:delay 1s;
 #/ip hotspot user set email="wifi@local"  [find name=$HSuser];
 /system logging enable 0
 :log info "Revised and Modified By: fjoCharmedones"
}
}
#EOF

:if ($com!="") do={
/system logging enable 0
:log warning "Processing ON-Login event profile validation \n\r HotspotInterface: $interface \n\r Username: $user  "
/system logging disable 0;
	:local mac $"mac-address";
	:local macNoCol;
	:for i from=0 to=([:len $mac] - 1) do={ 
	  :local char [:pick $mac $i]
	  :if ($char = ":") do={
		:set $char ""
	  }
	  :set macNoCol ($macNoCol . $char)
	}
	
	:local validity [:pick $com 0 [:find $com ","]];
	
	:if ( $validity!="0m" ) do={
	:log warning "ProcessingOnUserProfile $user "
	/system logging disable 0
		:local sc [/sys scheduler find name=$user]; :if ($sc="") do={ :local a [/ip hotspot user get [find name=$user] limit-uptime]; :local validity [:pick $com 0 [:find $com ","]]; :local c ($validity); :local date [ /system clock get date]; /sys sch add name="$user" disable=no start-date=$date interval=$c on-event="/ip hotspot user remove [find name=$user]; /ip hotspot active remove [find user=$user]; /ip hotspot cookie remove [find user=$user]; /system sche remove [find name=$user]; /do { /file remove \"$hotspotFolder/data/$macNoCol.txt\";} on-error={} ;" policy=ftp,reboot,read,write,policy,test,password,sniff,sensitive,romon; :delay 2s; } else={ :local sint [/sys scheduler get $user interval]; :local validity [:pick $com  0 [:find $com ","]]; :if ( $validity!="" ) do={ /sys scheduler set $user interval ($sint+$validity); } };
		#:local sc [/sys scheduler find name=$user]; :if ($sc="") do={ :local a [/ip hotspot user get [find name=$user] limit-uptime]; :local c ($validity); :local date [ /system clock get date]; /sys sch add name="$user" disable=no start-date=$date interval=$c on-event="/ip hotspot user remove [find name=$user]; /ip hotspot active remove [find user=$user]; /file remove \"$hotspotFolder/data/$macNoCol.txt\"; /ip hotspot cookie remove [find user=$user]; /system sche remove [find name=$user]" policy=ftp,reboot,read,write,policy,test,password,sniff,sensitive,romon; :delay 2s; } else={ :local sint [/sys scheduler get $user interval]; :if ( $validity!="" ) do={ /sys scheduler set $user interval ($sint+$validity); } };
	}
/do {
/system logging enable 0
:log warning "ProcessingOnValiditySection"
/system logging disable 0;
:local validUntil [/sys scheduler get $user next-run];
/file print file="$hotspotFolder/data/$macNoCol" where name="noname.txt"; 
:delay 3s; 
/file set [find name="$hotspotFolder/data/$macNoCol.txt"] contents="";
:delay 1s;
/file set "$hotspotFolder/data/$macNoCol.txt" contents="$user#$validUntil";
:delay 1s;
/ip hotspot user set email=$HSemail  [find name=$HSuser];
} on-error={/system logging enable 0; 
:log error "ErrorOnValidityScriptSection"}

/do {	
/system logging enable 0
:log warning "ProcessingOnTelegramScriptSection"
/system logging disable 0;
:local infoArray [:toarray [:pick $com ([:find $com ","]+1) [:len $com]]];
:local mac $"mac-address"; 
:local totaltime [/ip hotspot user get $user limit-uptime]; 
:local date [ /system clock get date ]; 
:local time [/system clock get time ]; 
:local eg [/ip hotspot user get $user uptime]; 
:local rtime ($totaltime-$eg); 
:local expiry [ /sys sch get [/sys sch find where name=$user] next-run]; 
:local host [/ip dhcp-server lease get [ find mac-address=$mac ] host-name]; 
:local tu [ /ip hotspot user print count-only]; 
:local amt [:pick $infoArray 0];
:local ext [:pick $infoArray 1];
:local vendo [:pick $infoArray 2];
:local uactive [/ip hotspot active print count-only];
:local idle ( $tu - $uactive );
:local validUntil [/system scheduler get $user next-run];

:local getIncome [:put ([/system script get [find name=todayincome] source])];
/system script set source="$getIncome" todayincome;
:local getSales ($amt + $getIncome);
/system script set source="$getSales" todayincome;
:local getMonthlyIncome [:put ([/system script get [find name=monthlyincome] source])];
/system script set source="$getMonthlyIncome" monthlyincome;
:local getMonthlySales ($amt + $getMonthlyIncome);
/system script set source="$getMonthlySales" monthlyincome;
:local getLifetimeIncome [:put ([/system script get [find name=monthlyincome] source])];
/system script set source="$getMonthlyIncome" lifetimeincome;
:local getLifetimeSales ($amt + $getMonthlyIncome);
/system script set source="$getMonthlySales" lifetimeincome;
	
	:if ($enableTelegram=1) do={:local myhost; :local checkHostName [:put [:len [/ip dhcp-server lease get [ find mac-address=$mac ] host-name]]];
	     /system logging enable 0;
	     :log warning "SendingTelegramInfoSection"
		 /system logging disable 0;
	    :do {:set $myhost [/ip dhcp-server lease get [ find mac-address=$mac ] host-name];} on-error={:set $myhost "UNKnownHost";}
        :if ($checkHostName=0) do={:set $myhost "UNKnownHost"};		 
		:local vendoNew;
		:for i from=0 to=([:len $vendo] - 1) do={ 
		  :local char [:pick $vendo $i]
		  :if ($char = " ") do={
			:set $char "%20"
		  }
		  :set vendoNew ($vendoNew . $char)
		}
		/do {/tool fetch url="https://api.telegram.org/bot$telegramToken/sendmessage?chat_id=$chatId&text=<<======New Sales======>> %0A Vendo: $vendoNew %0A Voucher: $user %0A IP: $address %0A Device: $host %0A MAC: $mac %0A Amount: $amt %0A Total Time: $totaltime %0A Valid Until: $validUntil %0A Today Sales: $getSales %0A Monthly Sales : $getMonthlySales %0A Active Users: $uactive %0A <<=====================>>" keep-result=no;} on-error={/system logging enable 0; :log error "ErrorOnSendInfoTelegramSection"}
	}
} on-error={/system logging enable 0; 
:log error "ErrorOnTelegramScriptSection"}

};

:if ($enableRandomMacSyncFix=1) do={
/system logging enable 0
:log warning "ProcessingMACramdomizerScriptSection"
/system logging disable 0;
	:local cmac $"mac-address"
	:foreach AU in=[/ip hotspot active find user="$username"] do={
	  :local amac [/ip hotspot active get $AU mac-address];
	  :if ($cmac!=$amac) do={  /ip hotspot active remove [/ip hotspot active find mac-address="$amac"]; }
	}
};
/system logging enable 0
:log warning "LOGON-EVENT Finished Username: $user "
:log info  "Tested 30March2022 / hAPlite6.49.4 / Juanfi 3.2Wifi / 3.3 Script & Portal"
} on-error={/system logging enable 0;  :log error "MainScriptSectionError"}
#EndOfScript
~~~


