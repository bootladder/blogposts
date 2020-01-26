---
layout: post
title: 'Javascript Notes: The Relearning'
---
Learning to use this demo: https://github.com/muaz-khan/RecordRTC/blob/master/simple-demos/audio-recording.html  
Starting from the top... What is this?  `<meta name="viewport" content="width=device-width, initial-scale=1.0, minimum-scale=1.0">`  ah of course, it's for mobile devices.  https://stackoverflow.com/questions/14775195/is-the-viewport-meta-tag-really-necessary  
  
What is this?  `<div><audio controls autoplay></audio></div>`  It's the audio widget, which I did not know was an HTML tag.  `controls` shows the play, seek, volume controls, `autoplay` plays without pressing play.
  
What is this?  `<script src="https://webrtc.github.io/adapter/adapter-latest.js"></script>`   Oh... we need to use this adapter for cross browser compatibility, insulating us from name changes.  
  
Why does the demo code declare vars all over the place... Let's move them to the top.  Demo still works.  
  
So.. the control flow here is convoluted, impossible to read top to bottom.  Let's start from pressing the "Start Recording" button.  Using the inspector I see the button has a click event.  I see where this is set, and I'll move it to the top.  Ah finally some sense; all the click handlers are set in a block.  So I'll just move all the click handlers to the top.
  
Alright, the btnStartRecording.onclick.  This smells bad, I'm pretty sure?
```
btnStartRecording.onclick = function() {
    this.disabled = true;
    this.style.border = '';
    this.style.fontSize = '';

    if (!microphone) {
        captureMicrophone(function(mic) {
            microphone = mic;

// Different behavior for Safari?  Changes button style, does an alert?
            if(isSafari) {
                replaceAudio();

                audio.muted = true;
                setSrcObject(microphone, audio);
                audio.play();

                btnStartRecording.disabled = false;
                btnStartRecording.style.border = '1px solid red';
                btnStartRecording.style.fontSize = '150%';

                alert('Please click startRecording button again. First time we tried to access your microphone. Now we will record it.');
                return;
//Return from calling function inside callback?  What?
            }

//Calling this function again, using a helper named click which,
//instead of calling the function, sends another click event?
//Hmm, maybe because we want the call to happen asynchronous to this
//function call we're currently in, as opposed to recursive?
            click(btnStartRecording);
        });
        return;
    }
console.log("\ndid we get here?\n");
    replaceAudio();

    audio.muted = true;
    setSrcObject(microphone, audio);
    audio.play();

    var options = {
        type: 'audio',
        numberOfAudioChannels: isEdge ? 1 : 2,
        checkForInactiveTracks: true,
        bufferSize: 16384
    };

    if(navigator.platform && navigator.platform.toString().toLowerCase().indexOf('win') === -1) {
        options.sampleRate = 48000; // or 44100 or remove this line for default
    }

    if(recorder) {
        recorder.destroy();
        recorder = null;
    }

    recorder = RecordRTC(microphone, options);

    recorder.startRecording();

    btnStopRecording.disabled = false;
    btnDownloadRecording.disabled = true;
};
```
What happens when I Do Not Allow access to the microphone?  The console.log does not get logged.  
  
First I refresh the page. Then click the button.  The microphone is null so captureMicrophone() is called.  captureMicrophone() sets the Release Button (a global variable) to not disabled.  captureMicrophone() then checks that same microphone global, which is still null at this time.  Then tries to getUserMedia(), and since I Do Not Allow, we get the alert.  Ah.. at this point the function returns, but Does Not call the Callback!  So, the Start Recording click handler returns, and that's the end of the story.  

Now, what happens when I Allow Access to the microphone?  
Here we see the real magic, navigator.mediaDevices.getUserMedia() , which appears to take some configuration object?  On success the callback is called, with the mic being passed in from getUserMedia?  Then in the callback, if it's safari the user is required to press the button again, but otherwise the click() helper function generates another click event.  This time, the global variable microphone is not-null, so we go on to the end of the function where the recording actually happens.  
  
replaceAudio() , don't know why this is necessary, it replaces the <audio> tag with a new one, with optional src to be specified.  Why not just use the existing one without replacing it? 
  
setSrcObject() is a RecordRTC function, does it attach the microphone to the <audio> widget?  and then audio.play() moves the seeker and changes the play button to a pause button.  audio.muted=true is there, to stop playback during record, which would cause feedback.
  
Then, if there's a recorder it is destroyed, and then we create the RecordRTC object and start recording.

# note, what is navigator?
The navigator object contains information about the browser.  
In this demo we see
```
navigator.platform        == 'win'
navigator.mediaDevices
navigator.mediaDevices.getUserMedia
navigator.userAgent       == 'Safari'
navigator.msSaveOrOpenBlob
navigator.msSaveBlob
```
  
The Stop Recording Button click handler calls recorder.stopRecording, which will then call the stopRecordingCallback() which calls replaceAudio() and then audio.play()
  
# Random notes
`getUserMedia` is a promise, either calling a callback or an error.  Apparently, if the callback itself fails execution, then the error is caught.  Also, if this happens, the UserMedia is not released.
  
`https://www.youtube.com/watch?v=4ba0G8FQt5M`  This talk is good.
#Blobs
1.  FileReader can read content from a blob.  readAsText() , readAsArrayBuffer().
2.  A File is a Blob.
3.  FileReader can readAsText(myFile);
4.  ArrayBuffer is like a Blob, but the bytes are on disk.
5.  var bytes = new Uint8Array(buffer); //creates a typed array from an array buffer.
