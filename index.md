# Mux for Craft CMS: Documentation Review

Prepared for Rocket Park by Mumpe Cydrone, October 2026

## Why I did this

I am applying for the Technical Documentation Intern role. Instead of only telling you I can write documentation, I read the public docs for your Mux plugin as a new user would and noted where I got stuck or had to guess. I also wrote one new page to help with the biggest gap. These are fresh eyes notes, not criticism. The docs are already well organised.

## What already works well

- The sidebar is logical. Get Started, Feature Tour, Mux Element and Developers match how people actually move through a plugin.
- The webhook section lists every event the plugin handles, so nobody has to guess.
- Environment variables are recommended for credentials, with a full example block and sensible defaults.
- The feature page uses screenshots to show the edit screen, the tracks tab and the data tab.

## Gaps and proposed fixes

### 1. The Composer package name does not match Packagist

The Installation page shows `composer require rocketpark/craft-mux`. Packagist lists the package as `rocket-park/craft-mux`. If the second is correct, the first command fails for anyone who copies it.

**Fix:** Confirm the correct name and use it in the install command everywhere it appears.

Tested on a fresh Craft 5.11.5.1 install. The command from the docs, composer require rocketpark/craft-mux, failed with: Could not find a matching version of package rocketpark/craft-mux. The command composer require rocket-park/craft-mux installed version 2.6.0 on the first try. After that, plugin/list showed the handle as mux, and plugin/install with a capital M failed with: No plugin exists with the handle "Mux". The lowercase handle worked. The docs' own handle, craft-mux, failed too: plugin/install craft-mux returned No plugin exists with the handle "craft-mux". The working command is plugin/install mux. The docs join both commands with &&, so a reader who hits the first failure never reaches the second.

### 2. The webhook section ends at "Save and Test" without a test

The last step says to click Create Webhook, then stops. There is no way for a reader to know the webhook is working. Mux also cannot reach a site running on localhost, which catches almost everyone who follows the guide on a local setup.

**Fix:** Add a short verification step: upload a small video, then check that its status moves from processing to ready in Craft. Add a note about using a public tunnel when testing locally.

### 3. There is no troubleshooting page

The docs explain the happy path well. Nothing covers failed uploads, a video stuck on processing, secure playback errors, or the upload CORS setting for sites behind a reverse proxy.

**Fix:** Add one Troubleshooting page, organised by symptom. I have drafted it and attached it with this review.

### 4. The environment variable block carries a deprecated setting

The settings list marks MP4 Support as deprecated and points to Static Renditions. The example block still includes `MUX_MP4_SUPPORT` with no note.

**Fix:** Remove it from the example, or add a one line deprecation note beside it.

### 5. Variables are listed but not connected to the settings screen

The page recommends environment variables, then shows the settings fields separately. A new reader may not know how to point a field at a variable.

**Fix:** Add one example showing a settings field referencing an environment variable, in the way Craft expects.

Tested: typing $MUX\_TOKEN\_ID and $MUX\_TOKEN\_SECRET into the two token fields saved with no error. Each field says it can begin with an environment variable, but neither that hint nor the docs show the syntax.

### 6. The licensing link points to Craft 4 documentation

The plugin targets Craft 5, but the licensing link goes to the 4.x plugin docs.

**Fix:** Update the link to the current Craft documentation.

### 7. The dashboard steps read as blocks of text

In the Mux account and token sections, several separate actions run together in one line, such as signing up, verifying an email and logging in. On screen they read as one dense paragraph.

**Fix:** Turn each action into its own numbered step. Consider splitting the page into three: Install, Connect your Mux account, Set up webhooks. Mux account creation is Mux's own process and can link out.

### 8. The token permission names do not match Mux’s own

The setup page tells readers to give Mux Video “Full Access” and Mux Data “Read”. Mux’s documentation describes separate Read and Write permissions for each area, such as Mux Video Read and Write. It also says creating signing keys for secure playback needs System Write, which the setup page does not mention. I did not test the System Write point.

**Fix:** Use Mux’s exact permission names (Mux Video Read and Write, Mux Data Read) and add a note that Secure Playback with signed keys may need System Write.

### 9. A failed upload gives no reason

My test upload failed. The panel showed a red icon with only Retry and Remove, and no reason. The browser console showed a 500 error from the plugin’s upload action. The cause only appeared in Craft’s log, where Mux had answered the upload request with 401 Unauthorized, which means it rejected the access token. I have not confirmed what was wrong with my token, so I am reporting the missing message rather than the cause.

**Fix:** Show the reason in the panel when it is known, for example that Mux rejected the access token, and add this symptom to the Troubleshooting page below.

### 10. Several settings differ from how the docs describe them

Observed on the settings screen, not tested:

- Maximum Upload File Size shows 716800. The docs’ example uses 734003200 and says it is in bytes. Mux’s own uploader measures this limit in kilobytes, and 716800 kB is 700 MB.
- Upload Chunk Size is a dropdown set to 1 MB. The docs describe a default of 8 MB and a multiple of 256 KB rule.
- Max Resolution Tier shows 2160p, while the field’s help text and the docs say the default is 1080p.
- Static Renditions is a list of resolution checkboxes. The docs describe a choice of whether to enable it.
- Secure Playback is a dropdown showing Public. The docs say to enable it.
- The settings screen has captions options (default language and auto generate captions) that the setup page does not list.

**Fix:** Align field names, units and defaults between the screen and the docs, and state the unit on the field label.

## What I tested

Craft CMS 5.11.5.1, PHP 8.4.27, DDEV 1.25.5 with Docker Desktop and MariaDB 11.8, on Windows. Mux plugin version 2.6.0. So far I have tested the Composer install and the plugin install. I then created a Mux access token, saved it in the project’s .env file and referenced it from the plugin settings, which saved with no error. A test upload of a small MP4 then failed, which is covered in gap 9. I did not complete the webhook test or run the console commands.

## What is included

1. This review
2. A drafted Troubleshooting page, included in this document and ready for a VitePress docs folder
3. Links to my other documentation samples, in my application

## Drafted page: Troubleshooting

This is the page as I would hand it over for the docs folder. The upload entry comes from my own test. The rest follows your docs and Mux’s documentation, and I have not tested every entry.

### Quick checklist

1. Your Mux Token ID and Token Secret are saved in the plugin settings (MUX, then Settings, in the control panel sidebar).
2. The token has Mux Video Read and Write, and Mux Data Read.
3. A webhook exists in the same Mux environment as your token, and its URL matches the one shown in the plugin settings.
4. Your site is reachable from the public internet. Mux cannot send webhooks to localhost.
5. Your video file type is on the accepted extensions list.

### Upload problems

#### The upload fails with a red icon and a Retry button

The panel does not say why. Find the reason in two places.

1. Open your browser’s developer tools and go to the Network tab. Click Retry and look at the request to `actions/mux/assets/upload-asset`. A 500 status means the plugin hit an error on the server.
2. Open Craft’s web log at `storage/logs/web-YYYY-MM-DD.log` and search for `ERROR`.

If the log shows `MuxPhp\ApiException: [401]` and `401 Unauthorized`, Mux rejected your access token. Check that the Token ID and Secret are correct, that the token has not been revoked, and that it has Mux Video Read and Write. If you are not sure, generate a new token.

#### The file is rejected

The default maximum size is 700 MB. Only extensions on the accepted list can be uploaded from the control panel: MP4, MOV, MKV, WEBM, M4V, MP3, M4A, WAV, FLAC, AAC, OGG and OPUS. Add another type under Accepted Upload Extensions in the plugin settings.

If you changed the chunk size, use one of the options in the settings dropdown. If you set it through an environment variable, Mux requires a multiple of 256 KB.

#### The upload fails with a cross origin (CORS) error

This usually happens when the control panel sits behind a reverse proxy that changes the Host header. Set Upload CORS Origin to the public address of your control panel. If you are not behind a proxy, leave it empty.

### Webhook problems

#### A video stays on processing after it has uploaded

The plugin learns that a video is ready from Mux webhooks, so a webhook that never arrives leaves the status unchanged.

1. Copy the webhook URL from the plugin settings.
2. In the Mux dashboard, open Settings, then Webhooks, and confirm a webhook exists with exactly that URL.
3. Check the webhook is in the same Mux environment as your access token. Mux webhooks belong to one environment and do not fire for events in another.
4. Check the site is public. A site on localhost or a private address cannot receive webhooks, so on a local setup use a tunnel tool and put the tunnel address in the Mux webhook settings.
5. Look at the webhook’s recent deliveries in the Mux dashboard to see whether Mux got an error back from your site.

### Out of sync

#### Videos in Craft do not match videos in Mux

Run a full sync from the command line, then clear the plugin cache if the control panel still shows old information.

```bash
php craft mux/sync/all
php craft mux/cache/clear
```

### Playback problems

#### The video will not play on the front end

If Secure Playback is on, the player needs a signed token, so check that signed keys exist and that the token is generated for the video you are playing. Mux’s documentation says creating signing keys needs System Write on your access token. If you set domain or geographic restrictions, confirm the page you are testing from is allowed. To rule secure playback out, switch it off for a test video and try again.

### Captions and analytics

Automatic captions are generated after processing, so they can arrive later than the video. If a track shows an error, check the track details and the webhook delivery log in Mux. If the Data tab is empty, check that your access token has Mux Data Read, and remember that a new video needs real views before numbers appear.

### Still stuck?

Email support through the Support page or open an issue on GitHub. Include your Craft version, your plugin version, and what you see in the Mux webhook delivery log.

## Closing note

I would be glad to talk through any of this, or to turn the fixes into a pull request against the docs.

Mumpe Cydrone cydronem@gmail.com Portfolio: https://mumpe-cydrone-portfolio.vercel.app/ GitHub: https://github.com/Cydo73
