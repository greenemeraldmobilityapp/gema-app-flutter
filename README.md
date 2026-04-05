# GEMA App Flutter

A simple Flutter login app with a single login page and web preview support.

## Structure

- `pubspec.yaml` — Flutter package configuration
- `lib/main.dart` — application code with login page
- `web/index.html` — web entry point for browser preview

## Running Preview in Codespace

1. Make sure Flutter SDK is installed in the Codespace environment.
2. Open the terminal and run:

```bash
cd /workspaces/gema-app-flutter
flutter pub get
flutter run -d web-server --web-hostname 0.0.0.0 --web-port 8080
```

3. Use Codespace port forwarding to open `http://localhost:8080` from your browser.
4. On your Android tablet, open the forwarded web URL to preview the app.

> If Flutter is not installed yet, install Flutter SDK or enable the Flutter extension in Codespace.
