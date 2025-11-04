# Pickleball Watch Scorer Website

This directory contains a static landing page for the Pickleball Watch Scorer app. It is ready to be hosted with GitHub Pages.

## Structure

- `index.html` – main marketing page with feature overview, screenshots, and App Store details.
- `privacy.html` – required privacy policy for App Store submission.
- `styles.css` – shared styles for the site.

## Deploying with GitHub Pages

1. Push this repository to GitHub if you haven't already.
2. Open the repository on GitHub and go to **Settings → Pages**.
3. Under **Build and deployment**, set **Source** to `GitHub Actions`, then click **Save**.
4. Push the repository; the included `pages.yml` workflow will upload the `website/` directory and publish it to `https://<username>.github.io/<repo>/`.

Updates to any files in this directory will automatically trigger the GitHub Pages workflow and redeploy once pushed to `main`.
