# Adventure Lab Helper

A phone-friendly browser app for keeping Adventure Lab clues, your answer attempts, notes, and stop locations together. It stores data in the browser's local storage on your device and can be added to your Android home screen.

## Run with GitHub Codespaces

1. Open this repository: https://github.com/aaronatwake/adventure-lab-helper
2. In GitHub, choose **Code → Codespaces → Create codespace on main**. If Codespaces is available for your account, it will open a browser-based development environment; usage may depend on your account's included quota or billing settings.
3. In the Codespaces terminal, run:

   ```sh
   python3 -m http.server 8000 --bind 0.0.0.0
   ```

4. Open the **Ports** tab. For port `8000`, choose **Open in Browser** (or make the port public only if you intentionally want others to access it; the app itself has no server-side storage).
5. On Android Chrome, open the forwarded HTTPS address, then use **⋮ → Add to Home screen** or **Install app** if shown.

Keep the Codespace running while using the app. Your notes are stored in that browser's local storage, so they are tied to the device/browser and are not synced or backed up. Clearing site data can delete them. The location feature requires browser location permission and a secure context; the forwarded Codespaces URL should be HTTPS.

## Run locally

Serve this folder with any static web server, then open its address in your browser. For example:

```sh
python3 -m http.server 8000
```

## Features

- Add and remove stops
- Save clue text, answer attempts, and notes as you type
- Track how many stops have an answer
- Copy an answer to the clipboard
- Save current coordinates with browser geolocation
- Cache the app shell for returning visits

This independent note-taking helper does not connect to or submit answers to the Adventure Lab app.
