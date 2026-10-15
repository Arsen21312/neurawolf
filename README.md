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
