---
layout: post
title: 'Twilio, Tokbox, Vidyo:  Demos'
---
# Vidyo
1.  Vidyo PaaS does not include the Media Server (Media Bridge) as part of the platform.  Vidyo does provide a Media Bridge docker image, which the application can use.  `To record or live-stream multiple Vidyo conferences simultaneously, you need to create and deploy a separate Docker container for each recording and live stream.` .  Woah, weird use of Docker.  Using docker exec to control the media bridge.  Every recording needs to have a different config file, specifying the path and filename for the recording, plus other options.  What??  
  
Yeah... no thanks.
  
# Tokbox
1.  Hmm, it markets the ability to record and playback.  Mostly takes about "archiving" , hardly mentions playback at all.  Something's fishy about it.  
  
# Twilio
  
# Sinch?  Red5?
  
# Hmm, the server needs to be able to be a peer in a peer2peer connection.
This is the only way the server can stream back the recording.
Otherwise, the only way a client can play the recording is to download it.
Sure this could happen automatically with a AJAX call.  
Actually I just tried it, a HTML5 audio tag with the src pointing to a URL.  Works pretty good.
