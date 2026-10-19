# Bootstrap: Expo App

Use this guide to create an Android-first Expo app in a Yarn monorepo.

## Prerequisites

- Node.js 22.x and Yarn 1.22.x, as specified by the monorepo setup
- Expo/EAS account and Android build prerequisites when building locally
- Docker, if using the optional local EAS builder

## Workspace package

Create the app under `apps/<app-name>` and give it a scoped workspace name. For example:

```json
{
  "name": "@apps/my-app",
  "version": "1.0.0",
  "private": true,
  "scripts": {
    "start": "expo start",
    "android": "expo start --android",
    "ios": "expo start --ios",
    "web": "expo start --web",
    "android-build": "eas build -p android --profile preview",
    "android-build-local": "eas build -p android --profile development --local",
    "start-container": "docker compose up -d --build",
    "stop-container": "docker compose down",
    "build-container": "docker compose exec builder eas build -p android --profile development --local"
  },
  "dependencies": {
    "expo": "^57.0.2",
    "react-native": "0.86.0",
    "react-native-web": "^0.21.2"
  },
  "peerDependencies": {
    "react": "^19.2.4",
    "react-dom": "^19.2.4"
  },
  "devDependencies": {
    "@babel/core": "^8.0.1",
    "expo-dev-client": "~57.0.5",
    "expo-status-bar": "^57.0.0",
    "react-native-webview": "14.0.1"
  }
}
```

Keep dependency versions aligned with the Expo SDK selected for the new app. Install from the monorepo root so Yarn can resolve the workspace:

```bash
yarn install
yarn workspace @apps/my-app start
```

This package example does not define `build`, `lint`, or `test` scripts. Do not assume those tasks exist when wiring the app into Turbo.

## Files and structure

An initial app can use a small structure such as:

```text
apps/my-app/
├── App.js
├── app.json
├── eas.json
├── Dockerfile
├── docker-compose.yml
├── .prettierrc
├── package.json
└── yarn.lock
```

Use a unique workspace package name, Expo slug, Android application ID, and EAS project ID for each app.

## Expo configuration

Set the supported platforms and Android application ID in `app.json`. Keep any explicit SDK configuration in sync with the installed Expo version.

```json
{
  "expo": {
    "name": "My App",
    "slug": "my-app",
    "version": "1.0.0",
    "platforms": ["android"],
    "android": {
      "package": "com.example.myapp"
    }
  }
}
```

Use a new, globally unique reverse-domain Android package identifier for each published app.

## App entry and constants

To display a remote page in a React Native WebView, keep app-specific values such as the page URL in a named constant:

```js
import { StatusBar } from 'expo-status-bar';
import { View } from 'react-native';
import { WebView } from 'react-native-webview';

const APP_URL = 'https://example.com/';
const screenStyle = {
  flex: 1,
  paddingTop: 35,
  backgroundColor: '#d9d9d9',
};

export default function App() {
  return (
    <View style={screenStyle}>
      <StatusBar style="dark" />
      <WebView style={{ flex: 1 }} source={{ uri: APP_URL }} />
    </View>
  );
}
```

If URL handling grows beyond a constant, place it in a small utility and validate external input before passing it to the WebView. For example:

```js
export function isHttpUrl(value) {
  try {
    const url = new URL(value);
    return url.protocol === 'http:' || url.protocol === 'https:';
  } catch {
    return false;
  }
}
```

## EAS build profiles

`eas.json` sets the CLI version floor, uses remote app versioning, and defines preview, development, and production profiles. Adapt build profiles to the app's release workflow.

```json
{
  "cli": {
    "version": ">= 16.0.0",
    "appVersionSource": "remote"
  },
  "build": {
    "preview": {
      "developmentClient": true,
      "distribution": "internal"
    },
    "development": {
      "android": {
        "buildType": "apk"
      }
    },
    "production": {}
  }
}
```

Run workspace Android commands from the monorepo root:

```bash
yarn workspace @apps/my-app android
yarn workspace @apps/my-app android-build
yarn workspace @apps/my-app android-build-local
```

## Optional local EAS builder

For local Android builds in Docker, add the following files to the app workspace. Select an Android platform and build-tools version supported by the Expo SDK and native dependencies in use.

**`Dockerfile`**

```dockerfile
FROM node:22-bookworm

ENV ANDROID_HOME=/opt/android-sdk
ENV ANDROID_SDK_ROOT=/opt/android-sdk
ENV PATH="/opt/android-sdk/cmdline-tools/latest/bin:/opt/android-sdk/platform-tools:${PATH}"

ARG ANDROID_COMMAND_LINE_TOOLS=11076708
ARG ANDROID_PLATFORM=android-35
ARG ANDROID_BUILD_TOOLS=35.0.0

RUN apt-get update && apt-get install -y --no-install-recommends \
    openjdk-17-jdk \
    git \
    curl \
    unzip \
    && rm -rf /var/lib/apt/lists/*

RUN mkdir -p "${ANDROID_HOME}/cmdline-tools" \
    && curl -fsSL \
      "https://dl.google.com/android/repository/commandlinetools-linux-${ANDROID_COMMAND_LINE_TOOLS}_latest.zip" \
      -o /tmp/android-command-line-tools.zip \
    && mkdir -p /tmp/android-command-line-tools \
    && unzip -q /tmp/android-command-line-tools.zip -d /tmp/android-command-line-tools \
    && mv /tmp/android-command-line-tools/cmdline-tools "${ANDROID_HOME}/cmdline-tools/latest" \
    && rm -rf /tmp/android-command-line-tools /tmp/android-command-line-tools.zip \
    && yes | sdkmanager --licenses >/dev/null \
    && sdkmanager "platform-tools" "platforms;${ANDROID_PLATFORM}" "build-tools;${ANDROID_BUILD_TOOLS}"

RUN npm install --global eas-cli

WORKDIR /app
CMD ["sleep", "infinity"]
```

**`docker-compose.yml`**

```yaml
services:
  builder:
    build: .
    working_dir: /app
    volumes:
      - .:/app
      - ${HOME}/.eas:/root/.eas
      - ${HOME}/.expo:/root/.expo
    environment:
      EXPO_TOKEN: ${EXPO_TOKEN:-}
      EAS_NO_VCS: "1"
```

Start the builder and run a local EAS build with the package scripts:

```bash
export EXPO_TOKEN=your-token
yarn workspace @apps/my-app start-container
yarn workspace @apps/my-app build-container
yarn workspace @apps/my-app stop-container
```

Pass credentials through the environment; do not hard-code or commit tokens or `.env` files. The mounted Expo/EAS directories persist local state on the host. Update the Node, Android command-line tools, platform, and build-tools versions in the Dockerfile to versions compatible with the selected Expo SDK.

## Formatting

The example `.prettierrc` uses two-space indentation, single quotes, semicolons, and import sorting. The package manifest does not declare Prettier or the configured import-sorting plugin; add compatible development dependencies before relying on that formatter configuration.

## Utilities and tests

The package example has no utility module, test files, test script, or test-runner dependency. Keep the initial scaffold limited to the package's runtime needs. If a utility is added, add tests with a runner compatible with the selected Expo SDK and wire a `test` script before treating tests as runnable. An illustrative Jest-style test for the URL helper above is:

```js
import { isHttpUrl } from './isHttpUrl';

describe('isHttpUrl', () => {
  it('accepts HTTP and HTTPS URLs', () => {
    expect(isHttpUrl('https://example.com')).toBe(true);
    expect(isHttpUrl('http://example.com')).toBe(true);
  });

  it('rejects other protocols and malformed URLs', () => {
    expect(isHttpUrl('javascript:alert(1)')).toBe(false);
    expect(isHttpUrl('not a URL')).toBe(false);
  });
});
```

This test is an example only; do not add or invoke a test runner unless its compatible dependency and project configuration are installed.
