# Reliable install process for Battle.net under Lutris
Yes, I have repeated this process multiple times already (even additional times to check the install,) which includes removing all Lutris configuration and re-downloading all components.

## Note
This may no longer be necessary. I no longer use Battlenet, but this was a common question as it broke a lot.

## Steps

01. Go to Lutris.net, search for [battle.net](https://lutris.net/games/battlenet/) and and click install
02. Click install on the Lutris box. In the Blizzard installer, select language, click install
03. Uncheck "Launch battle net when you start your computer" and press continue
04. Close installer, then **wait** for Lutris to complete its tasks
05. **DO NOT CLICK LAUNCH** - Just close the Lutris install task
06. Right click on the battle net tile, and click configure. Change the runner to ge-proton
> *Info: if you don't have a "ge-proton" in runners, then click on the 3 bars top right -> preferences -> select the updates tab, and click "Check Again" on the wine version check box to trigger a download*
07. Close Lutris completely and restart
> *Info: any WINE games or other things (like foobar2000) you are running will pin Lutris "open" even though the UI closes. Please shut them down first.*
08. Open Lutris again. Run Battle.net
> *Info: if it is the first time using ge-proton please wait - it needs to install elements of sniper runtime from Valve*
09. When the Login box appears, click the cog and change your region to the correct one, and then login
> *Info: If the elements of the login box vanish, just wave your cursor over them to force them to redraw*
10. After login, click the "update and restart" box in the top right.
19. All done, ready to install games. not really a step, but you know what I mean.
