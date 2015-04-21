## Introduction ##

This display theme makes use of Microsoft Agent, if available, to speak any notifications.

## Requirements ##

Users will need to be running Microsoft Agent. There are various ways that they can obtain it, and in many cases it will already be installed as part of some other application. It's also possible with the correct licenses to distribute it from your own servers. The following describes the full steps to obtain the software:

  * If you are _not_ running Windows 2000 then you need to [install the Microsoft Agent core components](http://activex.microsoft.com/activex/controls/agent2/MSagent.exe);
  * If you are running Windows XP you need to [install the older Microsoft Speech API](http://activex.microsoft.com/activex/controls/sapi/spchapi.exe). This is because Windows XP ships with a speech API that is newer than the one Microsoft Agent uses;
  * If you do not see an entry for a Lernout & Hauspie text-to-speech (TTS) engine in 'Add or Remove Programs' (in your control panel), you will need to install one. You can start with [American English](http://activex.microsoft.com/activex/controls/agent2/tv_enua.exe) or [British English](http://activex.microsoft.com/activex/controls/agent2/lhttseng.exe). Other languages are available from the Microsoft Agent download page, linked to below;
  * If you want to be able to control the speech engines--such as how fast they speak, or how loud--then [install the Speech Control Panel](http://download.microsoft.com/download/c/9/e/c9ee5f5d-7631-4ee7-aee4-dbd22b2b1439/SpchCpl.exe).

More details and text-to-speech engines for various languages, are available at '[Microsoft Agent downloads for end-users](http://www.microsoft.com/msagent/downloads/user.asp)'.

## Testing ##

Once installed, notifications can be directed to MS Agent. Verify your installation using [this test](http://backplanejs.googlecode.com/hg/_samples/yowl/test-yowl-speech.html), which is set-up to use the British English TTS Engine (see bullet 3 above).