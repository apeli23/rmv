### Merge Videos with Next js

## Introduction

This article demonstrates how to merge two videos using nextjs.

## Codesandbox

The final demo on [Codesandbox](/).

<CodeSandbox
title="mergevideos"
id=" "
/>

You can get the project github repo using [Github](/).

## Prerequisites

Entry-level javascript and React/Nextjs knowledge.

## Setting Up the Sample Project

Use the command `npx create-next-app videomerge` to create a new Next.js project and head to the directory using `cd videomerge`

Download the necessary dependencies:

`npm install cloudinary`

We will begin by setting up our backend. Our backend will involve [Cloudinary](https://cloudinary.com/?ap=em) intergration for our media file upload.
.

Use this [Link](https://cloudinary.com/console) to create your cloudinary account and log into it. Use your dashboard to access your environment variables.

In your project root directory, create a new file named `.env`. Paste the following. Fill the blanks with your environment variables from cloudinary dashboard.

```
CLOUDINARY_CLOUD_NAME =

CLOUDINARY_API_KEY =

CLOUDINARY_API_SECRET=
```

Restart your project using `npm run dev`.

In the `pages/api` folder, create a new file named `upload.js`. We will use this for our backend intergration.

Start by configuring the environment keys and libraries. This avoids code duplication.

```
var cloudinary = require("cloudinary").v2;

cloudinary.config({
    cloud_name: process.env.CLOUDINARY_NAME,
    api_key: process.env.CLOUDINARY_API_KEY,
    api_secret: process.env.CLOUDINARY_API_SECRET,
});
```

Create a handler function to execute the POST request:

```
export default async function handler(req, res) {
    if (req.method === "POST") {
        let url = ""
        try {
            let fileStr = req.body.data;
            const uploadedResponse = await cloudinary.uploader.upload_large(
                fileStr,
                {
                    resource_type: "video",
                    chunk_size: 6000000,
                }
            );
            url = uploadedResponse.url
        } catch (error) {
            res.status(500).json({ error: "Something wrong" });
        }

        res.status(200).json({data: url});
    }
}
```

The function above receives media data from the frontend and uploads it to cloudinary. It also captures the media file's cloudinary link and stores it in the "url" variable. This variable is finaly sent back to front end as response

This concludes our backend. Let us now merge our videos.

### Front End

To merge two videos in nextjs, we will of course require two sample videos one of which for simplified demonstration will have to contain a major single colorfill, for example, a unique colored back ground like a green screen video. This move allows us to easily replace the green color with frames from the second video. For this article, we will use the two samples for our [foreground](https://res.cloudinary.com/dogjmmett/video/upload/v1644764254/foreground_vulcfz.mp4) and [background](https://res.cloudinary.com/dogjmmett/video/upload/v1644764315/background_wjdrt7.mp4) respectively. Lets begin.

Import the following hooks

```
import { useState, useRef, useEffect } from "react";
```

In the root function, we'll start by declaring our variables. We will use the criteria below:

i. foreground - we refference this to the foreground video DOM element
ii). background - well create a video element to play our background.
iii). canvas - we will merge our videos in this canvas
iV). context - this variable captures the foreground video frame as image and pass use the drawImage method to pass its video size
v). temporaryCanvas - we use this tro extract each frame like in the foreground
vi). temporaryContext - captures background video size like in the first video.
vi). link - state hook to contain the backend response link.
v). blob - state hook that will store chunks of processed blob to for video
upload.

Use the code below to implement the functions above

```
  let foreground, background, canvas, context, temporaryCanvas, temporaryContext;
  const canvasRef = useRef();
  const [link, setLink] = useState("");
  const [blob, setBlob] = useState();
```

Start by creating the video element and canvas in the root fuinction return statement.

```
return (
    <div>
      <div className="container">
        <div className="header">
          <h1 className="heading">
            <span onClick={computeFrame} className="heading-primary-main">
              <b>Merge videos with nextjs</b>
            </span>
          </h1>
        </div>
      </div>
      <div className="row">
        <div className="column">
          <video
            className="video"
            crossOrigin="Anonymous"
            src="[videos/foreground.mp4](https://res.cloudinary.com/dogjmmett/video/upload/v1644847286/foreground_z4ga7a.mp4"
            id="video"
            width="800"
            height="450"
            autoPlay
            muted
            loop
            type="video/mp4"
          />
        </div>
        <div className="column">
          {link ? (
            <a href={link}>LINK : {link}</a>
          ) : (
            <h3>your link will show here...</h3>
          )}
          <canvas
            className="canvas"
            ref={canvasRef}
            id="output-canvas"
            width="800"
            height="450"
          ></canvas>
          <br />
          <a
            href="#"
            className="btn btn-white btn-animated"
            onClick={uploadHandler}
          >
            Get video Link
          </a>
        </div>
      </div>
    </div>
  );

```

we will wrap our functions around a useEffect hook so the videos start processing when the page renders. Inside the hook, start by refferencing the video element and canvas.

```
foreground = document.getElementById("video");
canvas = document.getElementById("output-canvas");
context = canvas.getContext("2d");

```

create a video element for the background and let it play and loop in muted condition

```
    background = document.createElement("video");
    background.setAttribute("width", 800);
    background.setAttribute("height", 450);
    background.src = "videos/background.mp4";
    background.muted = true;
    background.autoplay = true;
    background.play();
    background.loop = true;
```

create the temporary canvas and refference its context. then play the fore ground using event listener as you run the compute frame function.

```
    temporaryCanvas = document.createElement("canvas");
    temporaryCanvas.setAttribute("width", 800);
    temporaryCanvas.setAttribute("height", 450);
    temporaryContext = temporaryCanvas.getContext("2d");
    foreground.addEventListener("play", computeFrame);
```

your useEffect should look as follows:

```
  useEffect(() => {
    foreground = document.getElementById("video");
    canvas = document.getElementById("output-canvas");
    context = canvas.getContext("2d");


    background = document.createElement("video");
    background.setAttribute("width", 800);
    background.setAttribute("height", 450);
    background.src = "videos/background.mp4";
    background.muted = true;
    background.autoplay = true;
    background.play();
    background.loop = true;


    temporaryCanvas = document.createElement("canvas");
    temporaryCanvas.setAttribute("width", 800);
    temporaryCanvas.setAttribute("height", 450);
    temporaryContext = temporaryCanvas.getContext("2d");
    foreground.addEventListener("play", computeFrame);
  }, []);
```

In the `computeFrame` function, we will start by directly putting the image data to the output canvas.

```
    temporaryContext.drawImage(foreground, 0, 0, foreground.width, foreground.height);
    let frame = temporaryContext.getImageData(0, 0, foreground.width, foreground.height);
```

Do the same for the background

```
      temporaryContext.drawImage(background, 0, 0, background.width, background.height);
    let frame2 = temporaryContext.getImageData(
      0,
      0,
      background.width,
      background.height
    );
```

The image data we created above is in a single array format which begins with the first row's pixel followed buy the next in the same row. It then begins with the same procedure with the next row untill the entire image is covered.
There are 4 pixels in each data. The first three are the RGB values and the last one is known as alpha. There will be 4 array spaces contained in each pixel. The final array size will be 4 times the actual pixel number.

Create a loop that checks all the RGB pixel values and multiply each pixel value by 4. We will use 0 as an offset for R which is the index value for all pixels . G and B wil need offset of 1 and 2 respectively.

```
    for (let i = 0; i < frame.data.length / 4; i++) {
      let r = frame.data[i * 4 + 0];
      let g = frame.data[i * 4 + 1];
      let b = frame.data[i * 4 + 2];
    }
```

We will then use an if statement to check each pixel close to green color and replace its RGB value with second video. That should merge the green screen background with the second video. In our we demo we showcase spider man in paris like below:

![Merged videos](https://res.cloudinary.com/dogjmmett/image/upload/v1645751585/merged_videos_brfwok.png "Merged Videos")

With our videos merged, we can capture the processed canvas using a media stream to chunks of blob and pass the blob to the blob state hook created ealier. Remember to use the timeout method to set up number of seconds you wish your chuncks to record.

```
    const chunks = [];
    const cnv = canvasRef.current;
    const stream = cnv.captureStream();
    const rec = new MediaRecorder(stream);
    rec.ondataavailable = e => chunks.push(e.data);
    rec.onstop = e => setBlob(new Blob(chunks, { type: 'video/webm' }));
    rec.start();
    setTimeout(() => rec.stop(), 10000);
  }
```

Finaly, use an async function `uploadHandler` to encode the blob to a base 64 string using a file reader and send the encoded file to the backend using a POST method. The upload function's response is then assigned to the link state hook created earlier which allows the link to be viewed from the UI as shown

![complete UI](https://res.cloudinary.com/dogjmmett/image/upload/v1645753714/merged_videos_bqc0rf.png "complete UI")

We have succesfuly merged 2 videos using only next js. Try this out to enjoy your experience. Happy coding