import os
import json
import asyncio
import threading
import queue
import pyperclip
import time
import pyaudio
from websockets.sync.client import connect

# AI_TTV API setup
RATE = 48000
DEFAULT_URL = f"wss://api.deepgram.com/v1/speak?encoding=linear16&sample_rate={RATE}"
DEFAULT_TOKEN = os.environ.get("API_KEY", None)

# Audio settings
FORMAT = pyaudio.paInt16
CHANNELS = 1
CHUNK = 8000

def main():
    print(f"Connecting to {DEFAULT_URL}")
    _socket = connect(
        DEFAULT_URL, additional_headers={"Authorization": f"Token {DEFAULT_TOKEN}"}
    )
    _exit = threading.Event()
    
    def receiver():
        speaker = Speaker()
        speaker.start()
        try:
            while not _exit.is_set():
                message = _socket.recv()
                if isinstance(message, bytes):
                    speaker.play(message)
        finally:
            speaker.stop()
    
    threading.Thread(target=receiver, daemon=True).start()
    
    prev_text = ""
    while not _exit.is_set():
        text_input = pyperclip.paste()
        if text_input and text_input != prev_text:
            prev_text = text_input
            print(f"Reading: {text_input}")
            _socket.send(json.dumps({"type": "Speak", "text": text_input}))
        time.sleep(1)

class Speaker:
    def __init__(self, rate=RATE, chunk=CHUNK, channels=CHANNELS):
        self._queue = queue.Queue()
        self._audio = pyaudio.PyAudio()
        self._stream = self._audio.open(
            format=pyaudio.paInt16,
            channels=channels,
            rate=rate,
            output=True,
            frames_per_buffer=chunk
        )
        self._exit = threading.Event()
        self._thread = threading.Thread(target=self._play, daemon=True)

    def start(self):
        self._exit.clear()
        self._thread.start()

    def stop(self):
        self._exit.set()
        self._thread.join()
        self._stream.stop_stream()
        self._stream.close()

    def play(self, data):
        self._queue.put(data)
    
    def _play(self):
        while not self._exit.is_set():
            try:
                data = self._queue.get(timeout=0.05)
                self._stream.write(data)
            except queue.Empty:
                pass

if __name__ == "__main__":
    main()
