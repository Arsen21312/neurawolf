# Neural Wise Wolf / ARS public media for social scheduling

Shared **public media hosting** for `@neural_wise_wolf` and `@arsizkrg`, used by ChatGPT + Zernio.

## The free publishing pipeline

1. Generate image (4:5 for Instagram feed) inside ChatGPT.
2. If needed, put the image automatically into the user's connected Google Drive folders: `NeuralWiseWolf-media` or `arsizkrg-media`. This is an intermediate transfer, not the final publishing URL.
3. Fetch the image bytes via the connected Google Drive action `fetch(download_raw_file=true, include_base64=true)`.
4. Upload binary to GitHub via `create_blob(content=<base64>, encoding="base64")`, then `create_tree` (base tree), `create_commit`, and `update_ref`. Always use the latest branch head for concurrency safety.
5. Use the resulting direct public URL in `Zernio.schedule_post`: `https://raw.githubusercontent.com/Arsen21312/neurawolf/main/<path>.png`.
6. Check `Zernio.get_post` after publishing. A saved/scheduled status does not mean Instagram successfully posted. Never duplicate already-published posts.

## Paths

- Neural Wise Wolf: `media/nww/YYYY-MM/<unique-name>.png`
- Arsentiy: `media/arsizkrg/YYYY-MM/<unique-name>.png`

## Safety and maintenance

- The GitHub repository is public. Store only social-post content intended for public publication, never private assets.
- Avoid `at.adobe.com` temporary previews and `drive.google.com` / `lh3.googleusercontent.com` links for Zernio: they previously failed with 403.
- A Zernio `validate_post` success is not a guarantee that Instagram will fetch the image. Verify actual `published` and `post_url`.
- This repository is not unlimited object storage. Monitor growth and consider a dedicated object storage service if the library approaches GitHub's recommended repository size.
- Never touch projects `Replita` or any existing Supabase databases when uploading these image assets.

## Recovery and scheduling runbook (tested 16 October 2026)

For both Instagram accounts `@neural_wise_wolf` and `@arsizkrg`, use the pipeline **ChatGPT → GitHub public media → Zernio → Instagram**. Metricool is not used for new scheduling.

- Accept `.png` and `.jpg` media in `media/nww/YYYY-MM/` and `media/arsizkrg/YYYY-MM/` with distinct names. Keep original artwork and caption paired.
- Upload image bytes to GitHub and confirm the blob exists on `main` before handing `https://raw.githubusercontent.com/Arsen21312/neurawolf/main/<path>` to Zernio. Preserve the file extension.
- For a future Zernio post with an old temporary Adobe URL, use `update_scheduled_post` to replace only its `media_urls`; keep its original publish time and caption. Verify `targets[].status == pending`.
- For a **failed** Zernio post, preserve the original content and account, create a **new** scheduled post with the permanent GitHub URL, confirm the new post is `scheduled` with `targets[].status == pending`, and **only then** cancel the old failed record. Updating the old failed record in place can leave `targets[].status == failed` even when the top-level status turns `scheduled`.
- Check all scheduled records for duplicate captions/targets and confirm no records remain `failed`. A scheduled state alone is not proof of Instagram publication. After the publish time, verify the `published` status and `post_url` in Zernio.
- Schedule in the intended local time for Karaganda, Kazakhstan (UTC+05:00, `Asia/Almaty`) and leave reasonable gaps between feed posts. Never repost something that is already live.
- Do not touch Replita or any existing Supabase project for this workflow.

On 16 October 2026, 5 failed posts were rescheduled with GitHub media, 4 at-risk future posts had their Adobe links replaced, and Zernio reported 9 scheduled posts with pending Instagram targets and 0 remaining failed records at the end of the repair.
