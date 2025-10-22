# Mobile Pipeline Demo - Compass Use Case

A production-ready CircleCI configuration demonstrating advanced mobile CI/CD capabilities for teams migrating from Bitrise or other platforms.

## Key Features Demonstrated

### 🚀 M4 Pro Performance
- Side-by-side M4 Pro vs M2 Pro builds showing 30-50% performance gains
- Both executors run in parallel for direct comparison
- Xcode 15.4.0 on iOS 17.5 simulators

### 🧪 Intelligent Test Parallelization
- **22 parallel test shards** with timing-based test splitting
- Automatically distributes tests across containers for optimal speed
- Addresses long-running UI test pain points

### 📦 Multi-App Support
- Matrix strategy for 4 apps (Compass iOS/Android, Glide iOS/Android)
- Shared module build supporting multiple apps
- Independent iOS and Android deployment pipelines

### 🛠️ No Forced Tooling
- Works with **Bazel** (not just Gradle)
- Custom CLI tools for deployment (no Fastlane required)
- Use your existing tools and scripts as-is

### 💰 Cost Optimization
- Right-sized resources (small for simple tasks, xlarge for builds)
- Efficient caching strategies
- Smart dependency graph to avoid unnecessary work

## Use Case: Compass Mobile Team

This demo addresses the specific needs of Compass, a real estate company evaluating CircleCI:

- **Current State:** 4 mobile apps, 20 developers, 4,500 builds/month on Bitrise
- **Pain Points:** UI tests taking hours with 22 shards, complex YAML, cost constraints
- **Solution:** M4 Pro performance, intelligent test splitting, platform unification

## Workflow Architecture

```
Shared Module Build
    ├── Android Build + Test ──────────┐
    ├── iOS Build (M4 Pro) ────┐       │
    └── iOS Build (M2 Pro) ────┤       │
                               │       │
    iOS UI Tests (22 shards) ◄─┘       │
            │                          │
            ├── Deploy Compass iOS     │
            ├── Deploy Glide iOS       │
            │                          │
            └──────────────────────────┴── Deploy Compass Android
                                        └── Deploy Glide Android
```

### Dependency Flow

1. **Shared Module Build** runs first (supports code reuse across apps)
2. **Platform Builds** run in parallel:
   - Android Build + Test
   - iOS Build (M4 Pro) 
   - iOS Build (M2 Pro)
3. **iOS UI Tests** run after both iOS builds complete (22 parallel shards with timing-based splitting)
4. **Deployments** run after their respective tests pass:
   - iOS apps deploy after iOS builds + UI tests
   - Android apps deploy after Android build + tests

## Configuration Highlights

### M4 Pro vs M2 Pro Comparison

```yaml
executors:
  ios-m4pro-medium:
    macos:
      xcode: "15.4.0"
    resource_class: m4pro.medium
  
  ios-m2pro-medium:
    macos:
      xcode: "15.4.0"
    resource_class: m2pro.medium
```

Both run the same build in parallel so you can directly compare timing.

### Parallel Test Splitting

```yaml
ios_ui_tests_parallel:
  executor: ios-m4pro-medium
  parallelism: 22
  steps:
    - run:
        name: Run UI tests with timing-based test splitting
        command: |
          # CircleCI automatically splits tests by historical timing data
          TESTFILES=$(find iosApp -name "*UITest.swift" | \
            circleci tests split --split-by=timings --timings-type=classname)
```

### Multi-App Matrix

```yaml
- deploy:
    name: "Deploy << matrix.app_name >>-ios"
    matrix:
      parameters:
        app_name: ["Compass", "Glide"]
        deploy_type: ["ios"]
```

Easily scales to support multiple apps without duplicating configuration.

### Custom Tooling Support

```yaml
- run:
    name: Build shared module (Bazel-compatible approach)
    command: |
      # Compass uses Bazel - CircleCI supports any build tool
      echo "Running: bazel build //shared:all --config=ios"
      # Demo uses Gradle but approach works identically with Bazel
      ./gradlew :shared:build :shared:test
```

## Quick Start

1. Fork this repo
2. Connect to CircleCI
3. Pipeline runs automatically on push
4. Compare M4 vs M2 Pro build times in the UI
5. Watch 22 parallel test shards distribute intelligently

## Real-World Impact

**Compass's Migration Goals:**
- Reduce 17-35 min build times with M4 Pro
- Fix UI test duration issues with intelligent splitting
- Unify platform (rest of org already on CircleCI)
- Stay under cost cap with optimized resources
- Use existing custom tools (no Fastlane rewrite)

**This Demo Shows How CircleCI Addresses Each Point**

## Repository Structure

```
mobile-pipeline-demo/
├── .circleci/
│   └── config.yml          # Main CircleCI configuration
├── androidApp/             # Android application code
├── iosApp/                 # iOS application code
├── shared/                 # Shared Kotlin Multiplatform code
└── README.md               # This file
```

## Key Configuration Decisions

1. **Why M4 and M2 both run?** To provide direct performance comparison during the demo
2. **Why 22 shards?** Matches Compass's current Bitrise setup for accurate comparison
3. **Why separate iOS/Android deploys?** Proper dependency isolation (iOS tests don't block Android deploys)
4. **Why timing-based splitting?** Most efficient distribution for real-world test suites with varied execution times

## Tags

`mobile-ci-cd` `ios` `android` `circleci` `m4-pro` `bazel` `parallel-testing` `compass-demo` `bitrise-migration`

## License

MIT License - See LICENSE file for details
