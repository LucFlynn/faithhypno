# VSL video files

Drop the two self-hosted videos here, then set their paths in the `VIDEOS` config
block near the bottom of `index-v3.html`:

    vsl-primary.mp4      hero video   ("Watch This Before You Try to Relax Again")
    vsl-primary.jpg      poster frame for it (1280x720, shown before play)
    vsl-secondary.mp4    mid-page video ("Why Fighting Every Thought Keeps the Loop Alive")
    vsl-secondary.jpg    poster frame for it

Encoding: H.264 / AAC, 1280x720, web-optimised (`-movflags +faststart`) so the
video starts playing before the whole file has downloaded. Example:

    ffmpeg -i source.mov -vf scale=1280:-2 -c:v libx264 -crf 23 -preset slow \
           -c:a aac -b:a 128k -movflags +faststart vsl-primary.mp4
    ffmpeg -ss 3 -i vsl-primary.mp4 -frames:v 1 -q:v 3 vsl-primary.jpg

Keep each file under ~100 MB. Vercel serves them as static assets; anything
larger belongs on Vercel Blob or a CDN, with the full URL used in the config.
