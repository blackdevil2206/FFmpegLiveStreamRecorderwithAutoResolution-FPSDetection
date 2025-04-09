This Python script checks whether a live stream is online, retrieves the video resolution and framerate using FFprobe, and then records the stream using FFmpeg. It includes video filtering, text overlay, and async audio resampling. If video details can't be fetched, it uses default recording settings. It also handles corrupt frames and includes a retry mechanism for offline streams.


import subprocess
import time

def get_video_details(url, ffprobe_path):
    """ Retrieves the resolution and framerate of a video stream using FFprobe. """
    command = [
        ffprobe_path,
        '-v', 'error',
        '-select_streams', 'v:0',
        '-show_entries', 'stream=width,height,r_frame_rate',
        '-of', 'csv=p=0',
        url
    ]
    try:
        result = subprocess.run(command, stdout=subprocess.PIPE, stderr=subprocess.PIPE, text=True, check=True)
        width, height, frame_rate = result.stdout.strip().split(',')
        frame_numerator, frame_denominator = map(int, frame_rate.split('/'))
        fps = frame_numerator / frame_denominator
        return int(width), int(height), fps
    except subprocess.CalledProcessError as e:
        print(f"FFprobe error: {e.stderr}")
        return None, None, None
    except ValueError:
        print("Error processing video details.")
        return None, None, None

def is_stream_online(url, ffmpeg_path):
    """ Checks if the stream is online using FFmpeg. """
    command = [ffmpeg_path, '-i', url, '-t', '5', '-f', 'null', '-']
    try:
        subprocess.run(command, stderr=subprocess.PIPE, stdout=subprocess.PIPE, check=True)
        return True
    except subprocess.CalledProcessError:
        return False
def record_stream(url, output_file, ffmpeg_path, resolution, fps):
    """ Records the video stream using FFmpeg with error handling for corrupt frames and timestamps. """

    font_path = "C\\:/Windows/Fonts/Arial.ttf"
    combined_video_filter = f"yadif,fps={fps},drawtext=text='Add your name':x=w-tw-10:y=h-th-10:fontfile='{font_path}':fontcolor=white@0.2:fontsize=16"

    command = [
        ffmpeg_path, 
      
        '-i', url,  # Input stream
        '-vf', combined_video_filter,  # Deinterlacing and text overlay
        '-c:v', 'libx264',
        '-c:a', 'aac',
        '-ac', '2',
        '-b:a', '128k',
        '-af', 'aresample=async=1',  # Fix audio sync issues
        '-threads', '6',
        '-preset', 'superfast',
        '-crf', '21',
        '-movflags', '+faststart',
        
        output_file
    ]
    
    try:
        subprocess.run(command, check=True)
        print(f"Recording complete. File saved as {output_file}")
    except subprocess.CalledProcessError as e:
        print(f"An error occurred during recording: {e}")



def generate_new_filename(base_name, extension):
    """ Generates a new filename with a timestamp. """
    timestamp = time.strftime("%Y%m%d-%H%M%S")
    return f"{base_name}_{timestamp}.{extension}"

if __name__ == "__main__":
    stream_url = r"Add your Link or File which you need convert from 25fps to 50fps"
    base_output_filename = "Video name"
    output_extension = "mp4"
    ffmpeg_executable_path = r"ffmpeg Folder"
    ffprobe_executable_path = r"ffmpeg Folder"

    while True:
        if is_stream_online(stream_url, ffmpeg_executable_path):
            width, height, fps = get_video_details(stream_url, ffprobe_executable_path)
            if width is None or height is None or fps is None:
                print("Failed to get video details, using default settings.")
                width, height, fps = 1920, 1080, 50  # Default settings
            else:
                # Set FPS to 50 if below 50; otherwise, use the source FPS
                if fps < 50:
                    fps = 50
            resolution = (width, height)
            output_filename = generate_new_filename(base_output_filename, output_extension)
            print(f"Stream is online. Recording to {output_filename} at resolution {resolution[0]}x{resolution[1]} and {fps} fps...")
            record_stream(stream_url, output_filename, ffmpeg_executable_path, resolution, fps)
        else:
            print("Stream is offline. Retrying in 10 seconds...")
            time.sleep(10)
