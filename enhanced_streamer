import pygame
from urllib.request import urlopen, Request
import threading
import time

class EnhancedStreamer:
    def __init__(self):
        pygame.mixer.init(frequency=22050, size=-16, channels=2, buffer=512)
        self.playlist = []
        self.current_index = 0
        self.volume = 0.5
        pygame.mixer.music.set_volume(self.volume)
        self.stream_file = None  # The open file-like object
        self.is_playing = False
        self.play_thread = None

    def load_playlist(self, urls):
        """Load a list of audio URLs into the playlist."""
        self.playlist = urls
        if self.playlist:
            print(f"Loaded playlist with {len(urls)} tracks.")

    def stream_and_play(self, url, start_byte=0):
        """Stream audio from URL using HTTP range requests if start_byte > 0.
        Loads progressively as Pygame reads—true streaming without full download.
        For seeking: Reload stream with new range (restarts playback from offset).
        Extension: For seamless seek, implement a custom file-like with buffer + range on seek() calls.
        """
        try:
            headers = None
            if start_byte > 0:
                headers = {'Range': f'bytes={start_byte}-'}
                print(f"Streaming from byte {start_byte}...")
            req = Request(url, headers=headers)
            self.stream_file = urlopen(req)
            pygame.mixer.music.load(self.stream_file)
            return True
        except Exception as e:
            print(f"Error streaming/loading: {e}")
            return False

    def play_current(self):
        """Play the current track in the playlist."""
        if not self.playlist:
            print("No playlist loaded.")
            return
        url = self.playlist[self.current_index]
        print(f"Playing track {self.current_index + 1}: {url}")
        if self.stream_and_play(url):
            self.play()
        else:
            print("Failed to load track.")

    def play(self):
        if self.stream_file and not self.is_playing:
            pygame.mixer.music.play()
            self.is_playing = True
            print("Playing... (Controls: p=pause/unpause, s=stop, n=next, b=back, +=vol up, -=vol down, q=quit)")
            # Start monitor thread
            self.play_thread = threading.Thread(target=self._monitor_controls)
            self.play_thread.daemon = True
            self.play_thread.start()

    def pause(self):
        if self.is_playing:
            pygame.mixer.music.pause()
            self.is_playing = False
            print("Paused.")

    def unpause(self):
        if not self.is_playing and self.stream_file:
            pygame.mixer.music.unpause()
            self.is_playing = True
            print("Resumed.")

    def stop(self):
        pygame.mixer.music.stop()
        self.is_playing = False
        if self.stream_file:
            self.stream_file.close()
            self.stream_file = None
        print("Stopped.")

    def next_song(self):
        """Switch to next track (cycles)."""
        if not self.playlist:
            return
        self.stop()
        self.current_index = (self.current_index + 1) % len(self.playlist)
        self.play_current()

    def prev_song(self):
        """Switch to previous track (cycles)."""
        if not self.playlist:
            return
        self.stop()
        self.current_index = (self.current_index - 1) % len(self.playlist)
        self.play_current()

    def adjust_volume(self, delta):
        """Adjust volume by delta (-1.0 to 1.0)."""
        self.volume = max(0.0, min(1.0, self.volume + delta))
        pygame.mixer.music.set_volume(self.volume)
        print(f"Volume: {self.volume:.1f}")

    def _monitor_controls(self):
        """Monitor keyboard inputs during playback."""
        while self.is_playing:
            try:
                user_input = input().lower().strip()
                if user_input == 'p':
                    if self.is_playing:
                        self.pause()
                    else:
                        self.unpause()
                elif user_input == 's':
                    self.stop()
                    break
                elif user_input == 'n':
                    self.next_song()
                    break  # Exit monitor, new play starts its own
                elif user_input == 'b':
                    self.prev_song()
                    break
                elif user_input == '+':
                    self.adjust_volume(0.1)
                elif user_input == '-':
                    self.adjust_volume(-0.1)
                elif user_input == 'q':
                    self.stop()
                    break
                # Example seek: 'j 10000' to jump to byte 10000 (restarts from there)
                elif user_input.startswith('j '):
                    try:
                        start_byte = int(user_input.split()[1])
                        self.stop()
                        url = self.playlist[self.current_index]
                        self.stream_and_play(url, start_byte=start_byte)
                        self.play()
                    except (ValueError, IndexError):
                        print("Usage: j <byte_offset>")
            except EOFError:
                break
            time.sleep(0.1)  # Avoid busy-wait

    def run(self):
        """Start the app with playlist."""
        # Example playlist: Add more free public URLs
        example_playlist = [
            "https://www2.cs.uic.edu/~i101/SoundFiles/BabyElephantWalk60.wav",  # ~30s WAV
            "https://www.soundjay.com/misc/sounds/bell-ringing-05.wav",  # Free bell WAV (if accessible)
            # Add more: e.g., "https://freesound.org/data/previews/..." 
        ]
        self.load_playlist(example_playlist)
        if self.playlist:
            self.play_current()
        try:
            while True:
                time.sleep(1)
        except KeyboardInterrupt:
            self.stop()
            pygame.quit()

if __name__ == "__main__":
    streamer = EnhancedStreamer()
    streamer.run()
