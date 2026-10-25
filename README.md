# StreamLoop — all project files

A small full-stack dashboard to upload a video and send it repeatedly to an RTMP destination (such as YouTube Live) using FFmpeg.

## Important limitations
- GitHub Pages can host only the static dashboard (`public/`). It cannot run Node.js, store a 2GB upload, or keep FFmpeg running 24/7.
- Run the backend on an always-on server/VPS/cloud instance. The server needs enough disk space for the uploaded video and FFmpeg installed.
- A running FFmpeg process does not guarantee YouTube has accepted the stream. Confirm the preview/status in YouTube Studio.
- This starter app keeps the video list in server memory. After a server restart, upload the video again; files remain on disk but the in-memory list is not persisted.
- Keep `.env` private and never commit real credentials or stream keys.

## Requirements
- Node.js 20+
- FFmpeg installed and available as `ffmpeg` (`ffmpeg -version` should work)
- Enough server disk space for the video plus temporary space

## Run locally
1. Install Node.js and FFmpeg on the server/computer.
2. Copy `.env.example` to `.env` and adjust settings if needed.
3. Run `npm install`.
4. Run `npm start`.
5. Open `http://localhost:8080`.

For public use, deploy the Node backend to a VPS/cloud service that permits long-running processes and RTMP egress. Configure HTTPS and firewall rules. Some hosting providers terminate background processes or restrict outbound RTMP; verify before choosing one.

## Optional S3-compatible storage
Fill in `S3_ENDPOINT`, `S3_REGION`, `S3_BUCKET`, `S3_ACCESS_KEY_ID`, and `S3_SECRET_ACCESS_KEY` in `.env`. The backend keeps a local copy for FFmpeg and also uploads a copy to S3. The server must still have enough local disk space. Do not expose S3 credentials in frontend files.

## GitHub Pages (dashboard only)
Publish the contents of `public/` as the site root. In the dashboard's **Backend API URL** field, enter the public HTTPS URL of your separately deployed backend. Configure CORS/authentication and HTTPS properly before exposing the API to the internet.

## API
- `GET /api/health`
- `GET /api/videos`
- `POST /api/upload` — multipart form field `video`
- `POST /api/start` — JSON `{ "videoId": "...", "rtmpUrl": "rtmps://a.rtmps.youtube.com/live2", "streamKey": "...", "loop": true }`
- `POST /api/stop`
- `GET /api/status`

## 2GB uploads
The backend defaults to 2 GiB (`MAX_UPLOAD_BYTES=2147483648`). A reverse proxy/load balancer in front of Node must also allow bodies of at least this size and long upload timeouts. For example, with Nginx, set `client_max_body_size 2g;` in the relevant server/location block and tune read/send timeouts. Mobile network interruptions can still fail because this starter uses a single multipart request, not resumable chunk uploads.

## Security before production
This is a starter, not a hardened public service. Add authentication, per-user quotas, upload validation/scanning, HTTPS, rate limiting, log redaction, and protected admin endpoints before exposing it publicly. Anyone who can reach an unprotected `/api/start` endpoint could attempt to use your server to stream. Keep it behind authentication/firewall until secured.
