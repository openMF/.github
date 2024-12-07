## Kotlin Multiplatform Project Setup Guide

### Supported Platforms
- Android
- iOS
- Desktop (Windows, macOS, Linux)
- Web (Kotlin/JS, wasmJS)

### Recommended Project Structure
```
project-root/
│
├── buildLogic/            # Shared build configuration
├── gradle/                # Gradle wrapper and configuration
│
├── core/                  # Core business logic module
│   ├── common/            # Common code shared across platforms
│   ├── model/             # Model classes and data structures
│   ├── data/              # Data models and repositories
│   ├── network/           # Networking and API clients
│   ├── domain/            # Domain-specific logic
│   ├── ui/                # UI components and screens
│   ├── designsystem/      # App-wide design system
│   └── datastore/         # Local data storage
│
├── feature/               # Feature Specific module
│   ├── feature-a/         # Feature-specific logic
│   ├── feature-b/         # Feature-specific logic
│   └── feature-c/         # Feature-specific logic
│
├── androidApp/            # Android-specific implementation
├── iosApp/                # iOS-specific implementation
├── desktopApp/            # Desktop application module
├── webApp/                # Web application module
│
├── shared/                # Shared Kotlin Multiplatform code
│   ├── src/
│   │   ├── commonMain/    # Shared business logic
│   │   ├── androidMain/   # Android-specific code
│   │   ├── iosMain/       # iOS-specific code
│   │   ├── desktopMain/   # Desktop-specific code
│   │   ├── jsMain/        # Web-specific code
│       └── wasmJsMain/    # Web-specific code
│
├── Fastfile              # Fastlane configuration
├── Gemfile               # Ruby dependencies
└── fastlane/             # Fastlane configurations
```

### Development Environment
- JDK 17 or higher
- Kotlin 1.9.x
- Gradle 8.x
- Android Studio Hedgehog or later
- Xcode 15+ (for iOS development)
- Node.js 18+ (for web development)


### Fastlane Setup

#### Install Fastlane
```bash
# Install Ruby (if not already installed)
brew install ruby

# Install Fastlane
gem install fastlane

# Create Gemfile
bundle init

# Add Fastlane to Gemfile
bundle add fastlane
```

#### Fastfile Configuration
`Fastfile`:
```ruby
default_platform(:android)

platform :android do
  desc "Deploy internal tracks to Google Play"
  lane :deploy_internal do
    supply(
      track: 'internal',
      aab: 'mifospay-android/build/outputs/bundle/prodRelease/mifospay-android-prod-release.aab',
      skip_upload_metadata: true,
      skip_upload_images: true,
      skip_upload_screenshots: true,
    )
  end

  desc "Promote internal tracks to beta on Google Play"
  lane :promote_to_beta do
    supply(
      track: 'internal',
      track_promote_to: 'beta',
      skip_upload_changelogs: true,
      skip_upload_metadata: true,
      skip_upload_images: true,
      skip_upload_screenshots: true,
    )
  end

  desc "Promote beta tracks to production on Google Play"
  lane :promote_to_production do
    supply(
      track: 'beta',
      track_promote_to: 'production',
      skip_upload_changelogs: true,
      sync_image_upload: true,
    )
  end

  desc "Upload Android application to Firebase App Distribution"
  lane :deploy_on_firebase do
    release = firebase_app_distribution(
        app: "1:728434912738:android:0490c291986f0a691a1dbb",
        service_credentials_file: "mifospay-android/firebaseAppDistributionServiceCredentialsFile.json",
        release_notes_file: "mifospay-android/build/outputs/changelogBeta",
        android_artifact_type: "APK",
        android_artifact_path: "mifospay-android/build/outputs/apk/prod/release/mifospay-android-prod-release.apk",
        groups: "mifos-wallet-testers"
    )
  end

end


platform :ios do
    desc "Build iOS application"
    lane :build_ios do
        build_ios_app(
            project: "mifospay-ios/iosApp.xcodeproj/project.pbxproj",
            # Set configuration to debug for now
            configuration: "Debug",
            output_directory: "mifospay-ios/",
            output_name: "mifospay-ios-app"
        )
    end

    desc "Upload iOS application to Firebase App Distribution"
    lane :deploy_on_firebase do
        increment_build_number(
          xcodeproj: "mifospay-ios/iosApp.xcodeproj/project.pbxproj"
        )

        build_ios_app(
            project: "mifospay-ios/iosApp.xcodeproj/project.pbxproj",
            # Set configuration to debug for now
            configuration: "Debug",
        )
        release = firebase_app_distribution(
            app: "1:728434912738:ios:86a7badfaed88b841a1dbb",
            service_credentials_file: "mifospay-android/firebaseAppDistributionServiceCredentialsFile.json",
            release_notes_file: "mifospay-android/build/outputs/changelogBeta",
            groups: "mifos-wallet-testers"
        )

    end
end

```

### Code Quality Checks
- Static code analysis (Detekt)
- Code formatting (Spotless)
- Dependency guard
- Unit and UI testing

### Release Management
- Semantic versioning
- Automated beta deployments
- Cross-platform release automation

### Resources
- [Kotlin Multiplatform Documentation](https://kotlinlang.org/docs/multiplatform.html)
- [Compose Multiplatform Guide](https://www.jetbrains.com/lp/compose-multiplatform/)
- [Fastlane Documentation](https://docs.fastlane.tools/)

### Required GitHub Secrets

Configure the following secrets in your repository settings:

#### Android Secrets

- `ORIGINAL_KEYSTORE_FILE`: Base64 encoded release keystore
- `ORIGINAL_KEYSTORE_FILE_PASSWORD`: Keystore password
- `ORIGINAL_KEYSTORE_ALIAS`: Keystore alias
- `ORIGINAL_KEYSTORE_ALIAS_PASSWORD`: Keystore alias password
-
- `UPLOAD_KEYSTORE_FILE`: Base64 encoded release keystore
- `UPLOAD_KEYSTORE_FILE_PASSWORD`: Keystore password
- `UPLOAD_KEYSTORE_ALIAS`: Keystore alias
- `UPLOAD_KEYSTORE_ALIAS_PASSWORD`: Keystore alias password
-
- `GOOGLESERVICES`: Google Services JSON content
- `PLAYSTORECREDS`: Play Store service account credentials
- `FIREBASECREDS`: Firebase App Distribution credentials

#### iOS Secrets

- Notarization Credentials:
    - `NOTARIZATION_APPLE_ID`
    - `NOTARIZATION_PASSWORD`
    - `NOTARIZATION_TEAM_ID`


## Choosing and using a workflow template

1. On GitHub, navigate to the main page of the repository.

2. Under your repository name, click  Actions.
   Screenshot of the tabs for the "github/docs" repository. The "Actions" tab is highlighted with an orange outline.

3. If you already have a workflow in your repository, click New workflow.

4. The "Choose a workflow" page shows a selection of recommended workflow templates. 
   Find the workflow template that you want to use, then click Configure. 
   To help you find the workflow template that you want, you can search for keywords or filter by category.

5. If the workflow template contains comments detailing additional setup steps, follow these steps.

6. There are guides to accompany many of the workflow templates for building and testing projects. For more information, see "Building and testing."

7. Some workflow templates use secrets. For example, ${{ secrets.npm_token }}. If the workflow template uses a secret, store the value described in the secret name as a secret in your repository. For more information, see "Using secrets in GitHub Actions."

8. Optionally, make additional changes. For example, you might want to change the value of on to change when the workflow runs.

9. Click Start commit.

10. Write a commit message and decide whether to commit directly to the default branch or to open a pull request.