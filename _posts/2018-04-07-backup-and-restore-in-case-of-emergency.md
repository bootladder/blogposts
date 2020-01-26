---
layout: post
title: Cell Phone - Backup and Restore My Contacts!
---
I dropped my phone and it stopped working.  After a short existential crisis, 
I went to the screen repair place the next day.  It was replaced in an hour,
fully functional.  
I got lucky that time.  Eventually I won't be so lucky.  
**My contacts were not backed up.  I need to be more careful.**
  
# Grabbing .vcf's off my phone.
* First, why the hell are we using this ridiculous format?  
* Second, why is it so confusing to access the filesystem on my Android phone?  
  
I plug my phone into my Ubuntu laptop's USB.  Figuring out how to mount it was frustrating.  
But fortunately, Nautilus File Manager somehow knew how to mount it.  
Having no idea where it was mounted, I right clicked in Nautilus and selected "Open in a Terminal" or whatever.  
Then I did a `find . -name \*.vcf` to find the contacts.  I also exported my contacts in my "Phone App".  
Shouldn't the contacts just.. be there?  Instead of having to export them?  Yeah yeah, proprietary format.  
  
Now let's look at the .vcf files.  
I open one up in VIM.  Then go to the end of the file.  The person I see there, is the last person I added to my phone.  
Scrolling up, the contacts appear to be in chronological order.  
Scrolling back to the top, I see... totally random contacts.  Some from 10 years ago, some from 5 years ago.
Those defintely aren't in chronological order.  Perhaps they got mixed up since I've changed phones a couple times?  
  
Well, the reason I'm saying this is I have a problem.  I don't know if all of my contacts are there, or just some of them.  
For example, there are contacts stored on my SIM card.  Are they in the .vcf file?  
Also, I had a couple other .vcf files on my phone, don't remember when why or how, but they're there.
Weirdly, the old ones are larger in size than this last one I just exported!  WHY????  
Oh by the way, there are no dates for when the contacts were added.  I just had to guess by the person's name and where 
I was living when I met them, when the contact was added.
  
Whatever, it appears to be all of the contacts.  I'll just grab all the .vcf files off the phone.  
Also,  the contacts were exported to the SD card, not the phone flash.  
**So, if I just make a point to export my contacts every X days, then they'll be in the SD card which means I can grab them in the event that the phone dies for real**  
# Fortunately, this is all I need to do.  

