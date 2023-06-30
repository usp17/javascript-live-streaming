# Build a JavaScript Live Streaming App with Video SDK
### Introduction
This tutorial is about creating interactive live streaming app using [JavaScript](https://en.wikipedia.org/wiki/JavaScript) with Video SDK. It includes JavaScript, [Video SDK](https://www.videosdk.live/) and a server for streaming.

You should learn this tutorial to enhance your understanding of [live streaming](https://en.wikipedia.org/wiki/HTTP_Live_Streaming) and to gain practical skills in creating interactive live streams. The benefits of using this software configuration include the ability to create dynamic streaming experiences, engage with viewers in real-time and customize the streaming functionality according to specific needs.

By following this tutorial, you will set up a server, integrate the Video SDK with JavaScript and create interactive live streams. You will have a hands-on experience in building and testing a streaming application, allowing you to understand the process from start to finish.

At the end of this tutorial, you will have acquired new skills in video streaming and JavaScript programming. You will be able to create your own interactive live streams, customize streaming features, and deploy video streaming applications with the Video SDK and JavaScript.

## Prerequisites

- Video SDK Developer Account (Not having one? follow [Video SDK Dashboard](https://app.videosdk.live/))
- Have [Node](https://en.wikipedia.org/wiki/Node.js) and NPM installed on your device.

**Install Video SDK**​

You can install the Video SDK using below-mentioned npm command. Make sure you are in your app directory before running this command.

```js
npm install @videosdk.live/js-sdk
```

## Structure of the project​
Your project structure should look like this.

```js
  root
   ├── index.html
   ├── config.js
   ├── index.js
```
We are going to work on three files:

- index.html   : Responsible to create basic UI.
- config.js      : Responsible to store token.
- index.js       : Responsible to render meeting view and join the meeting.

## Step 1: Create UI of your project
In this step, We are going to create [HTML](https://en.wikipedia.org/wiki/HTML) file which will have two screens join-screen and grid-screen. Copy the below content and paste in your index.html file.

```js
<!DOCTYPE html>
<html>
  <head> </head>

  <body>
    <div id="join-screen">
      <!-- Create new Meeting Button -->
      <button id="createMeetingBtn">Create Meeting</button>
      OR
      <!-- Join existing Meeting -->
      <input type="text" id="meetingIdTxt" placeholder="Enter Meeting id" />
      <button id="joinHostBtn">Join As Host</button>
      <button id="joinViewerBtn">Join As Viewer</button>
    </div>

    <!-- for Managing meeting status -->
    <div id="textDiv"></div>

    <div id="grid-screen" style="display: none">
      <!-- To Display MeetingId -->
      <h3 id="meetingIdHeading"></h3>
      <h3 id="hlsStatusHeading"></h3>

      <div id="speakerView" style="display: none">
        <!-- Controllers -->
        <button id="leaveBtn">Leave</button>
        <button id="toggleMicBtn">Toggle Mic</button>
        <button id="toggleWebCamBtn">Toggle WebCam</button>
        <button id="startHlsBtn">Start HLS</button>
        <button id="stopHlsBtn">Stop HLS</button>
      </div>

      <!-- render Video -->
      <div id="videoContainer"></div>
    </div>
    <script src="https://sdk.videosdk.live/js-sdk/0.0.67/videosdk.js"></script>
    <script src="config.js"></script>
    <script src="index.js"></script>

    <!-- hls lib script  -->
    <script src="https://cdn.jsdelivr.net/npm/hls.js"></script>
  </body>
</html>
```

## Step 2: Implement Join Screen of your project
Set token (that you have generated from [Video SDK Dashboard](https://app.videosdk.live/)) in config.js file.

```js
// Auth token we will use to generate a meeting and connect to it
TOKEN = "Your_Token_Here";
```
Now get all the elements from DOM and declare following variables in index.js file and then add Event Listener to the join and create meeting buttons.
The join screen will work as medium to either schedule new meeting or to join existing meeting as a host or as a viewer.

These will have 3 buttons:
1. Join as a Host       : When this button is clicked, the person will join the entered meetingId as a HOST.
2. Join as a Viewer   : When this button is clicked, the person will join the entered meetingId as a VIEWER.
3. New Meeting        : When this button is clicked, the person will join a new meeting as HOST.

```js
// getting Elements from Dom
const joinHostButton = document.getElementById("joinHostBtn");
const joinViewerButton = document.getElementById("joinViewerBtn");
const leaveButton = document.getElementById("leaveBtn");
const startHlsButton = document.getElementById("startHlsBtn");
const stopHlsButton = document.getElementById("stopHlsBtn");
const toggleMicButton = document.getElementById("toggleMicBtn");
const toggleWebCamButton = document.getElementById("toggleWebCamBtn");
const createButton = document.getElementById("createMeetingBtn");
const videoContainer = document.getElementById("videoContainer");
const textDiv = document.getElementById("textDiv");
const hlsStatusHeading = document.getElementById("hlsStatusHeading");

// declare Variables
let meeting = null;
let meetingId = "";
let isMicOn = false;
let isWebCamOn = false;

const Constants = VideoSDK.Constants;

function initializeMeeting() {}

function createLocalParticipant() {}

function createVideoElement() {}

function createAudioElement() {}

function setTrack() {}

// Join Meeting As Host Button Event Listener
joinHostButton.addEventListener("click", async () => {
  document.getElementById("join-screen").style.display = "none";
  textDiv.textContent = "Joining the meeting...";

  roomId = document.getElementById("meetingIdTxt").value;
  meetingId = roomId;

  initializeMeeting(Constants.modes.CONFERENCE);
});

// Join Meeting As Viewer Button Event Listener
joinViewerButton.addEventListener("click", async () => {
  document.getElementById("join-screen").style.display = "none";
  textDiv.textContent = "Joining the meeting...";

  roomId = document.getElementById("meetingIdTxt").value;
  meetingId = roomId;

  initializeMeeting(Constants.modes.VIEWER);
});

// Create Meeting Button Event Listener
createButton.addEventListener("click", async () => {
  document.getElementById("join-screen").style.display = "none";
  textDiv.textContent = "Please wait, we are joining the meeting";

  const url = `https://api.videosdk.live/v2/rooms`;
  const options = {
    method: "POST",
    headers: { Authorization: TOKEN, "Content-Type": "application/json" },
  };

  const { roomId } = await fetch(url, options)
    .then((response) => response.json())
    .catch((error) => alert("error", error));
  meetingId = roomId;

  initializeMeeting(Constants.modes.CONFERENCE);
});

```

## Step 3: Initialize the meeting
Initialize the meeting by mode passed to the function and only create a local participant stream when mode is "CONFERENCE".

```js
// Initialize meeting
function initializeMeeting(mode) {
  window.VideoSDK.config(TOKEN);

  meeting = window.VideoSDK.initMeeting({
    meetingId: meetingId, // required
    name: "Thomas Edison", // required
    mode: mode,
  });

  meeting.join();

  meeting.on("meeting-joined", () => {
    textDiv.textContent = null;

    document.getElementById("grid-screen").style.display = "block";
    document.getElementById(
      "meetingIdHeading"
    ).textContent = `Meeting Id: ${meetingId}`;

    if (meeting.hlsState === Constants.hlsEvents.HLS_STOPPED) {
      hlsStatusHeading.textContent = "HLS has not stared yet";
    } else {
      hlsStatusHeading.textContent = `HLS Status: ${meeting.hlsState}`;
    }

    if (mode === Constants.modes.CONFERENCE) {
      // we will pin the local participant if he joins in `CONFERENCE` mode
      meeting.localParticipant.pin();

      document.getElementById("speakerView").style.display = "block";
    }
  });

  meeting.on("meeting-left", () => {
    videoContainer.innerHTML = "";
  });

  meeting.on("hls-state-changed", (data) => {
    //
  });

  if (mode === Constants.modes.CONFERENCE) {
    // creating local participant
    createLocalParticipant();

    // setting local participant stream
    meeting.localParticipant.on("stream-enabled", (stream) => {
      setTrack(stream, null, meeting.localParticipant, true);
    });

    // participant joined
    meeting.on("participant-joined", (participant) => {
      if (participant.mode === Constants.modes.CONFERENCE) {
        participant.pin();

        let videoElement = createVideoElement(
          participant.id,
          participant.displayName
        );

        participant.on("stream-enabled", (stream) => {
          setTrack(stream, audioElement, participant, false);
        });

        let audioElement = createAudioElement(participant.id);
        videoContainer.appendChild(videoElement);
        videoContainer.appendChild(audioElement);
      }
    });

    // participants left
    meeting.on("participant-left", (participant) => {
      let vElement = document.getElementById(`f-${participant.id}`);
      vElement.remove(vElement);

      let aElement = document.getElementById(`a-${participant.id}`);
      aElement.remove(aElement);
    });
  }
}
```

## Step 4: Controls for the Speaker
Next step is to create SpeakerView and Controls components to manage features such as join, leave, mute and unmute.
You'll get all the participants from meeting object and filter them for the mode set to CONFERENCE so only Speakers are shown on the screen.

```js
// leave Meeting Button Event Listener
leaveButton.addEventListener("click", async () => {
  meeting?.leave();
  document.getElementById("grid-screen").style.display = "none";
  document.getElementById("join-screen").style.display = "block";
});

// Toggle Mic Button Event Listener
toggleMicButton.addEventListener("click", async () => {
  if (isMicOn) {
    // Disable Mic in Meeting
    meeting?.muteMic();
  } else {
    // Enable Mic in Meeting
    meeting?.unmuteMic();
  }
  isMicOn = !isMicOn;
});

// Toggle Web Cam Button Event Listener
toggleWebCamButton.addEventListener("click", async () => {
  if (isWebCamOn) {
    // Disable Webcam in Meeting
    meeting?.disableWebcam();

    let vElement = document.getElementById(`f-${meeting.localParticipant.id}`);
    vElement.style.display = "none";
  } else {
    // Enable Webcam in Meeting
    meeting?.enableWebcam();

    let vElement = document.getElementById(`f-${meeting.localParticipant.id}`);
    vElement.style.display = "inline";
  }
  isWebCamOn = !isWebCamOn;
});

// Start Hls Button Event Listener
startHlsButton.addEventListener("click", async () => {
  meeting?.startHls({
    layout: {
      type: "SPOTLIGHT",
      priority: "PIN",
      gridSize: "20",
    },
    theme: "LIGHT",
    mode: "video-and-audio",
    quality: "high",
    orientation: "landscape",
  });
});

// Stop Hls Button Event Listener
stopHlsButton.addEventListener("click", async () => {
  meeting?.stopHls();
});
```
## Step 5: Speaker Media Elements
In this step, You will have create a function that helps you create audio and video elements to display local and remote participants for the speaker. You will also have to set the appropriate media track based on whether it's a video or audio.

```js
// creating video element
function createVideoElement(pId, name) {
  let videoFrame = document.createElement("div");
  videoFrame.setAttribute("id", `f-${pId}`);

  //create video
  let videoElement = document.createElement("video");
  videoElement.classList.add("video-frame");
  videoElement.setAttribute("id", `v-${pId}`);
  videoElement.setAttribute("playsinline", true);
  videoElement.setAttribute("width", "300");
  videoFrame.appendChild(videoElement);

  let displayName = document.createElement("div");
  displayName.innerHTML = `Name : ${name}`;

  videoFrame.appendChild(displayName);
  return videoFrame;
}

// creating audio element
function createAudioElement(pId) {
  let audioElement = document.createElement("audio");
  audioElement.setAttribute("autoPlay", "false");
  audioElement.setAttribute("playsInline", "true");
  audioElement.setAttribute("controls", "false");
  audioElement.setAttribute("id", `a-${pId}`);
  audioElement.style.display = "none";
  return audioElement;
}

// creating local participant
function createLocalParticipant() {
  let localParticipant = createVideoElement(
    meeting.localParticipant.id,
    meeting.localParticipant.displayName
  );
  videoContainer.appendChild(localParticipant);
}

// setting media track
function setTrack(stream, audioElement, participant, isLocal) {
  if (stream.kind == "video") {
    isWebCamOn = true;
    const mediaStream = new MediaStream();
    mediaStream.addTrack(stream.track);
    let videoElm = document.getElementById(`v-${participant.id}`);
    videoElm.srcObject = mediaStream;
    videoElm
      .play()
      .catch((error) =>
        console.error("videoElem.current.play() failed", error)
      );
  }
  if (stream.kind == "audio") {
    if (isLocal) {
      isMicOn = true;
    } else {
      const mediaStream = new MediaStream();
      mediaStream.addTrack(stream.track);
      audioElement.srcObject = mediaStream;
      audioElement
        .play()
        .catch((error) => console.error("audioElem.play() failed", error));
    }
  }
}
```

## Step 6: Implementing ViewerView
When the host starts live streaming, viewers will be able to see it.
To implement player view, You need to use hls.js. It will be helpful to play hls stream. You have already added script of hls.js in index.html file. Now on the hls-state-changed event, when participant mode is set to VIEWER and the status of hls is HLS_PLAYABLE, Pass the downstreamUrl to the hls.js and play it.

```js
// Initialize meeting
function initializeMeeting() {
  // ...

  // hls-state-chnaged event
  meeting.on("hls-state-changed", (data) => {
    const { status } = data;

    hlsStatusHeading.textContent = `HLS Status: ${status}`;

    if (mode === Constants.modes.VIEWER) {
      if (status === Constants.hlsEvents.HLS_PLAYABLE) {
        const { downstreamUrl } = data;
        let video = document.createElement("video");
        video.setAttribute("width", "100%");
        video.setAttribute("muted", "false");
        // enableAutoPlay for browser autoplay policy
        video.setAttribute("autoplay", "true");

        if (Hls.isSupported()) {
          var hls = new Hls();
          hls.loadSource(downstreamUrl);
          hls.attachMedia(video);
          hls.on(Hls.Events.MANIFEST_PARSED, function () {
            video.play();
          });
        } else if (video.canPlayType("application/vnd.apple.mpegurl")) {
          video.src = downstreamUrl;
          video.addEventListener("canplay", function () {
            video.play();
          });
        }

        videoContainer.appendChild(video);
      }

      if (status === Constants.hlsEvents.HLS_STOPPING) {
        videoContainer.innerHTML = "";
      }
    }
  });
}
```

You're done with the implementation of customized live streaming app in JavaScript using Video SDK.

## Conclusion

- You have configured a server and integrated the Video SDK with JavaScript to create interactive live streams.
- You have gained practical skills in live streaming and JavaScript programming.
- You can now create your own customized streaming experiences, engage with viewers in real-time and deploy video streaming applications.
- Go ahead and create advanced features like real-time messaging, screen-sharing and others. Browse our [documentation](https://docs.videosdk.live/javascript/guide/video-and-audio-calling-api-sdk/quick-start-ILS).
- If you face any problem, Feel free to join our [Discord community](https://discord.gg/Gpmj6eCq5u).

## More JavaScript Resources
- [JavaScript Audio/Video call documentation](https://docs.videosdk.live/javascript/guide/video-and-audio-calling-api-sdk/quick-start)
- [JavaScript interactive live stream documentation](https://docs.videosdk.live/javascript/guide/video-and-audio-calling-api-sdk/quick-start-ILS)
- [Code sample of JavaScript Video call](https://github.com/videosdk-live/videosdk-rtc-javascript-sdk-example)
- [Build JavaScript Video calling app](https://www.videosdk.live/blog/video-calling-javascript)
