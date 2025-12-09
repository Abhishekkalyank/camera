import asyncio
import cv2
import numpy as np
from fastapi import FastAPI, Request
from fastapi.responses import HTMLResponse
from fastapi.websockets import WebSocket
from aiortc import RTCPeerConnection, RTCSessionDescription, VideoStreamTrack
from aiortc.contrib.media import MediaRelay

app = FastAPI()
relay = MediaRelay()


class WebRTCVideoTrack(VideoStreamTrack):
    def __init__(self, track):
        super().__init__()
        self.track = relay.subscribe(track)

    async def recv(self):
        frame = await self.track.recv()
        img = frame.to_ndarray(format="bgr24")

        # 👇 You can process frame here using OpenCV
        # Example: draw line
        cv2.line(img, (0, 0), (img.shape[1], img.shape[0]), (0, 255, 0), 2)

        new_frame = frame.from_ndarray(img, format="bgr24")
        new_frame.pts = frame.pts
        new_frame.time_base = frame.time_base
        return new_frame


@app.post("/offer")
async def offer(request: Request):
    data = await request.json()
    offer = RTCSessionDescription(
        sdp=data["sdp"],
        type=data["type"]
    )

    pc = RTCPeerConnection()

    @pc.on("track")
    def on_track(track):
        if track.kind == "video":
            pc.addTrack(WebRTCVideoTrack(track))

    await pc.setRemoteDescription(offer)
    answer = await pc.createAnswer()
    await pc.setLocalDescription(answer)

    return {
        "sdp": pc.localDescription.sdp,
        "type": pc.localDescription.type,
    }
