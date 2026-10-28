# Troubleshooting

Something not working? Find your symptom below. Each section says what to check first and how to fix it.

   > **Start here.** Most problems come from one of four places: your Mux access token, your webhook, your upload settings, or secure playback. Work through the checklist first.

## Quick checklist

1. Your Mux Token ID and Token Secret are saved in the plugin settings (MUX, then Settings, in the control panel sidebar).
2. The token has Mux Video Read and Write, and Mux Data Read.
3. A webhook exists in the same Mux environment as your token, and its URL matches the one shown in the plugin settings.
4. Your site is reachable from the public internet. Mux cannot send webhooks to `localhost`.
5. Your video file type is on the accepted extensions list.

## Upload problems

### The upload fails with a red icon and a Retry button

The panel does not say why. Find the reason in two places.

1. Open your browser's developer tools and go to the Network tab. Click Retry and look at the request to `actions/mux/assets/upload-asset`. A 500 status means the plugin hit an error on the server.
2. Open Craft's web log at `storage/logs/web-YYYY-MM-DD.log` and search for `ERROR`.

If the log shows `MuxPhp\ApiException: [401]` and `401 Unauthorized`, Mux rejected your access token. Check that the Token ID and Secret are correct, that the token has not been revoked, and that it has Mux Video Read and Write. If you are not sure, generate a new token.

### The file is rejected

The default maximum size is 700 MB. Only extensions on the accepted list can be uploaded from the control panel: MP4, MOV, MKV, WEBM, M4V, MP3, M4A, WAV, FLAC, AAC, OGG and OPUS. Add another type under **Accepted Upload Extensions** in the plugin settings.

If you changed the chunk size, use one of the options in the settings dropdown. If you set it through an environment variable, Mux requires a multiple of 256 KB.

### The upload fails with a cross origin (CORS) error

This usually happens when the control panel sits behind a reverse proxy that changes the Host header. Set **Upload CORS Origin** to the public address of your control panel. If you are not behind a proxy, leave it empty.

## Webhook problems

### A video stays on "processing" after it has uploaded

The plugin learns that a video is ready from Mux webhooks, so a webhook that never arrives leaves the status unchanged.

1. Copy the webhook URL from the plugin settings.
2. In the Mux dashboard, open Settings, then Webhooks, and confirm a webhook exists with exactly that URL.
3. Check the webhook is in the same Mux environment as your access token. Mux webhooks belong to one environment and do not fire for events in another.
4. Check the site is public. A site on localhost or a private address cannot receive webhooks, so on a local setup use a tunnel tool and put the tunnel address in the Mux webhook settings.
5. Look at the webhook's recent deliveries in the Mux dashboard to see whether Mux got an error back from your site.

## Out of sync

### Videos in Craft do not match videos in Mux

Run a full sync from the command line, then clear the plugin cache if the control panel still shows old information.

```bash
php craft mux/sync/all
php craft mux/cache/clear
```

## Playback problems

### The video will not play on the front end

If Secure Playback is on, the player needs a signed token, so check that signed keys exist and that the token is generated for the video you are playing. Mux's documentation says creating signing keys needs System Write on your access token. If you set domain or geographic restrictions, confirm the page you are testing from is allowed. To rule secure playback out, switch it off for a test video and try again.

## Captions and analytics

Automatic captions are generated after processing, so they can arrive later than the video. If a track shows an error, check the track details and the webhook delivery log in Mux. If the Data tab is empty, check that your access token has Mux Data Read, and remember that a new video needs real views before numbers appear.

## Still stuck?

Email support through the [Support page](/support) or open an issue on [GitHub](https://github.com/rocketpark/craft-mux/issues). Include your Craft version, your plugin version, and what you see in the Mux webhook delivery log.
