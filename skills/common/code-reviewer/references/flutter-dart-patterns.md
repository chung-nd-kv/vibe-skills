# Flutter & Mobile Code Review Checklist

Apply only when reviewing Flutter/Dart code.

## Mobile (General)

- Handles app lifecycle events (background/foreground, termination)
- Avoids blocking the main/UI thread for I/O or heavy work
- Uses background work APIs appropriately
- Handles offline/poor network conditions gracefully
- Respects platform permissions and privacy prompts
- Optimizes image/video loading and caching
- Avoids excessive memory usage (large bitmaps, retained views)
- Battery and network usage are reasonable
- Analytics/logging do not leak PII

## Flutter

- Widget rebuilds are minimized; `const` used where possible
- State management is consistent with app conventions
- Avoid Riverpod/Bloc/GetX unless explicitly requested
- `BuildContext` usage is safe (no async gaps after dispose)
- `setState` guarded by `mounted`
- Avoids rebuilding expensive widgets without `const`/`keys`
- `ListView`/`GridView` use builders and proper keys
- Uses `SizedBox`/`Expanded`/`Flexible` appropriately to avoid layout thrash
- `Stream`/`Future`/`Controller` subscriptions are disposed
- `TextEditingController`/`FocusNode`/`AnimationController` disposed
- Uses `compute`/isolates for heavy work when needed
- Platform channels handle errors and threading correctly
- Avoid expensive work in `build()`
- Avoid `print`; use `dart:developer` `log` or `debugPrint`
- Enforce `flutter_lints` and `dart format`

## Dart (General)

- Prefer concise, declarative code and immutability
- Avoid `!` unless safety is guaranteed
- Prefer `Future/async/await` for async work and `Stream` for events
