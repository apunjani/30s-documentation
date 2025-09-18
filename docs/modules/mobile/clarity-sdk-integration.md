# Clarity SDK Integration Documentation

## Overview

This document provides a comprehensive guide to the Microsoft Clarity SDK integration in the Thirty Sundays Mobile Flutter application. Clarity is used for user behavior analytics and session recording to understand how users interact with the app.

## Dependencies

The project uses the `clarity_flutter` package version `^1.4.0` as specified in `pubspec.yaml`:

```yaml
dependencies:
  clarity_flutter: ^1.4.0
```

## Initialization

### 1. SDK Initialization Location

The Clarity SDK is initialized in the `AppLayoutWidget` class located at:
```
lib/common/widgets/app_layout_widget.dart
```

### 2. Initialization Process

The initialization happens when a user successfully authenticates. Here's the flow:

```dart
// In AppLayoutWidget.build()
ref.listen(
  authNotifier.select(
    (AuthState value) => value.userStateOrEmpty,
  ),
  (_, UserModel user) async {
    _initializeClarity(context, user);
    // ... other initialization code
  },
);
```

### 3. Initialization Method

```dart
void _initializeClarity(BuildContext context, UserModel user) async {
  // Only initialize in production mode
  if (!kReleaseMode || Environment.current != Env.production) {
    return;
  }

  // Set custom user ID if available
  if (user.customerId.isNotBlank()) {
    Clarity.setCustomUserId(user.customerId!);
  }

  // Configure and initialize Clarity
  final ClarityConfig config = ClarityConfig(
    projectId: 'tc9jcqy737',
    logLevel: LogLevel.None,
  );
  Clarity.initialize(context, config);
}
```

### 4. Configuration Details

- **Project ID**: `tc9jcqy737`
- **Log Level**: `LogLevel.None` (no debug logging)
- **Environment**: Only active in production mode
- **User Identification**: Uses the app's `customerId` as the custom user ID

## Usage

### 1. Screen Tracking

Screen navigation is automatically tracked through the `AppRouterObserver` class:

```dart
// lib/common/utils/app_router_observer.dart
class AppRouterObserver extends NavigatorObserver {
  @override
  void didPush(Route<dynamic> route, Route<dynamic>? previousRoute) {
    super.didPush(route, previousRoute);
    if (route.settings.name.isBlank()) {
      return;
    }

    // Track screen views in Clarity
    Clarity.setCurrentScreenName(route.settings.name!);
    
    // Also track in Firebase Analytics
    ref.read(analyticsService).logEvent(
      name: AnalyticsConstants.events.screenView,
      params: <String, String>{
        AnalyticsConstants.params.screenName: route.settings.name!,
      },
    );
  }
}
```

### 2. Router Integration

The `AppRouterObserver` is integrated into the app's routing system:

```dart
// lib/core/router/app_router.dart
final Provider<GoRouter> appRouter = Provider<GoRouter>((Ref ref) {
  return GoRouter(
    routes: $appRoutes,
    debugLogDiagnostics: kDebugMode,
    observers: <NavigatorObserver>[AppRouterObserver(ref)],
    // ... other configuration
  );
});
```

### 3. App Layout Integration

The `AppLayoutWidget` wraps the entire app and handles Clarity initialization:

```dart
// lib/app.dart
MaterialApp.router(
  title: '30 Sundays App',
  routerConfig: ref.read(appRouter),
  theme: AppTheme.lightTheme(context),
  builder: (_, Widget? child) {
    return AppLayoutWidget(
      child: child,
    );
  },
)
```

## Architecture

```
AppLayoutWidget (lib/common/widgets/app_layout_widget.dart)
├── Listens to authentication state changes
├── Initializes Clarity when user authenticates
└── Sets custom user ID

AppRouterObserver (lib/common/utils/app_router_observer.dart)
├── Monitors navigation events
├── Tracks screen views in Clarity
└── Logs events to Firebase Analytics

App Router (lib/core/router/app_router.dart)
└── Integrates AppRouterObserver as navigation observer
```

## Best Practices

1. **Environment Safety**: Clarity is only initialized in production to avoid unnecessary data collection during development
2. **User Privacy**: Custom user IDs are only set when explicitly available and not blank
3. **Error Handling**: Initialization is wrapped in environment checks to prevent issues
4. **Dual Tracking**: Both Clarity and Firebase Analytics are used for comprehensive user behavior analysis