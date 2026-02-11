> For the latest DRM service documentation, see the official TPStreams docs: [DRM Service | Streams Docs](https://developer.tpstreams.com/docs/drm-service/getting-started).

## TPStreams DRM-as-a-Service Sample

Sample integration showing how to use **TPStreams DRM-as-a-service** from a browser player, for the new `/drm_license` flow.

This repo contains:

- `drm_as_service_sample.html` – a standalone HTML page using **Video.js + videojs-contrib-eme** that:
  - Plays a Widevine-protected DASH stream.
  - Requests a license from TPStreams’ `/drm_license` API using the new `?data=<encoded_data>` format.

---

### How it works

1. **Encrypted stream**  
   The sample is preconfigured to use a test DASH manifest:  
   `https://dlbdnoa93s0gw.cloudfront.net/transcoded/taas-test/drm_true/video.mpd`

2. **Widevine license URL**  
   The license server URL is a TPStreams `/drm_license` endpoint generated with the new DRM-as-a-service flow:  
   `https://testing.tpstreams.com/api/v1/9q94nm/drm_license/?data=eyJjb250ZW50X2RhdGEiOiAiZXlKamIyNTBaVzUwWDJsa0lqb2dJbUZsWldFeU5HWm1NMlkxTVRSaU9URmhaakU0WWpneVptWTVNVFptTURObElpd2dJbVJ5YlY5MGVYQmxJam9nSW5kcFpHVjJhVzVsSWl3Z0ltUnZkMjVzYjJGa0lqb2dabUZzYzJWOSIsICJzaWduYXR1cmUiOiAidDhYMTVFTlZwMThyOHRGSDlNSzVyVFphZHhGWE1RdnVWbFRkV1ZQMndoRT0ifQ==`

   Inside the data blob, `content_data` contains:

   ```json
   {
     "content_id": "<uuid>",
     "download": false,
     "drm_type": "widevine"
   }
   ```

   and it is signed with the organization’s `drm_aes_signing_key` and `drm_aes_signing_iv`.

### Player behaviour

In `drm_as_service_sample.html`:

- Video.js is initialised with EME enabled.
- For the **Widevine** key system (`com.widevine.alpha`), we provide a custom `getLicense` function that:
  - Receives the Widevine key message (`keyMessage`) from the browser.
  - POSTs the `keyMessage` as `application/octet-stream` to the TPStreams license URL.
  - Passes the binary response back to the CDM as the license.
- For **FairPlay**, a similar `getLicense` hook is wired, but you’ll need to plug in a FairPlay-specific `/drm_license` URL generated with `"drm_type": "fairplay"`.

### Running the sample locally

Start any simple HTTP server in this repo directory, for example:

```bash
python -m http.server 8080
```

Then open the sample in your browser:

```text
http://localhost:8080/drm_as_service_sample.html
```

The page will:

- Load the test DASH manifest.
- Automatically request a Widevine license from the configured `/drm_license` URL.
- Play the decrypted content if the license request succeeds.

### Adapting to your environment

To adapt this sample for your own project:

1. **Generate your own license URL**  
   Use the logic from TPStreams `drm_api.md` (or a helper script) to build:

   ```text
   https://app.tpstreams.com/api/v1/<org_code>/drm_license/?data=<encoded_data>
   ```

   where `encoded_data` must contain:

   ```json
   {
     "content_data": "<base64 of {content_id, download, drm_type}>",
     "signature": "<AES signature over content_data>"
   }
   ```

2. **Update the constants in the HTML**  
   In `drm_as_service_sample.html`, change:

   - `dashUrl` to your MPD/HLS URL.
   - `widevineLicenseUrl` to your generated Widevine `/drm_license` URL.
   - (Optional) `fairplayLicenseUrl` to your FairPlay `/drm_license` URL.

3. **Secure it**  
   In a real application, you normally:

   - Generate the signed license URL on your backend, not in front-end code.
   - Pass that URL (or a short token) down to the client.

This sample is intentionally simple so you and your clients can see the end-to-end DRM flow with as little extra code as possible.
