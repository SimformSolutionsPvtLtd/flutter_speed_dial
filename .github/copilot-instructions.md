# Copilot Instructions for flutter_speed_dial

## Project Overview
`flutter_speed_dial` is a Flutter plugin that provides a **Material Design Speed Dial** (Floating Action Button with expandable menu). It's a customizable FAB that expands to reveal multiple action buttons, commonly used for quick access to related actions.

## Architecture

### Plugin Structure
```
flutter_speed_dial/
├── lib/
│   ├── flutter_speed_dial.dart         # Main export
│   └── src/
│       ├── speed_dial.dart             # Main Speed Dial widget
│       ├── speed_dial_child.dart       # Individual action button
│       ├── speed_dial_direction.dart   # Expansion direction enum
│       └── custom_scroll.dart          # Scroll behavior utilities
└── example/                            # Example app
```

### Component Hierarchy
```
SpeedDial (FAB Container)
  ├── Main FAB Button
  └── Expanded Menu
      ├── SpeedDialChild (Action 1)
      ├── SpeedDialChild (Action 2)
      └── SpeedDialChild (Action 3)
```

## Core API

### Basic Speed Dial
```dart
import 'package:flutter_speed_dial/flutter_speed_dial.dart';

class HomeScreen extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: Center(child: Text('Home')),
      floatingActionButton: SpeedDial(
        icon: Icons.add,
        activeIcon: Icons.close,
        children: [
          SpeedDialChild(
            child: Icon(Icons.photo),
            label: 'Add Photo',
            onTap: () => _addPhoto(),
          ),
          SpeedDialChild(
            child: Icon(Icons.video_call),
            label: 'Add Video',
            onTap: () => _addVideo(),
          ),
          SpeedDialChild(
            child: Icon(Icons.description),
            label: 'Add Document',
            onTap: () => _addDocument(),
          ),
        ],
      ),
    );
  }
}
```

### Customized Speed Dial
```dart
SpeedDial(
  // Main button
  icon: Icons.menu,
  activeIcon: Icons.close,
  backgroundColor: Colors.blue,
  foregroundColor: Colors.white,
  
  // Animation
  animatedIcon: AnimatedIcons.menu_close,
  animatedIconTheme: IconThemeData(size: 22.0),
  
  // Behavior
  visible: true,
  closeManually: false,
  curve: Curves.bounceIn,
  overlayColor: Colors.black,
  overlayOpacity: 0.5,
  
  // Tooltip
  tooltip: 'Quick Actions',
  heroTag: 'speed-dial-hero',
  
  // Expansion direction
  direction: SpeedDialDirection.up,
  
  // Children
  children: [
    SpeedDialChild(
      child: Icon(Icons.edit),
      backgroundColor: Colors.red,
      label: 'Edit',
      labelStyle: TextStyle(fontSize: 18.0),
      onTap: () => _onEdit(),
      onLongPress: () => _onEditLongPress(),
    ),
    // ... more children
  ],
  
  // Callbacks
  onOpen: () => print('OPENING DIAL'),
  onClose: () => print('DIAL CLOSED'),
  onPress: () => print('PRESSED'),
)
```

## SpeedDial Properties

### Main Button Styling
- **`icon`**: Icon shown when closed
- **`activeIcon`**: Icon shown when expanded
- **`animatedIcon`**: Animated icon (e.g., `AnimatedIcons.menu_close`)
- **`backgroundColor`**: Button background color
- **`foregroundColor`**: Icon color
- **`iconTheme`**: Custom icon theme
- **`elevation`**: Shadow elevation (default: 8.0)
- **`shape`**: Button shape (default: `CircleBorder()`)

### Animation & Behavior
- **`curve`**: Animation curve (default: `Curves.fastOutSlowIn`)
- **`direction`**: Expansion direction (`up`, `down`, `left`, `right`)
- **`closeManually`**: Require manual close (default: `false`)
- **`visible`**: Show/hide the speed dial (default: `true`)
- **`switchLabelPosition`**: Swap label position when direction changes

### Overlay
- **`overlayColor`**: Background overlay color when expanded
- **`overlayOpacity`**: Overlay transparency (0.0 - 1.0)
- **`renderOverlay`**: Whether to show overlay (default: `true`)

### Callbacks
- **`onOpen`**: Called when menu opens
- **`onClose`**: Called when menu closes
- **`onPress`**: Called when main button pressed

### Other
- **`tooltip`**: Tooltip text for main button
- **`heroTag`**: Hero animation tag (required if multiple FABs)
- **`spacing`**: Space between child buttons (default: 12.0)
- **`spaceBetweenChildren`**: Deprecated, use `spacing`

## SpeedDialChild Properties

### Appearance
- **`child`**: Widget displayed (usually an Icon)
- **`label`**: Text label shown next to button
- **`backgroundColor`**: Button background color
- **`foregroundColor`**: Icon/child color
- **`labelStyle`**: Text style for label
- **`labelBackgroundColor`**: Background color for label
- **`elevation`**: Shadow elevation
- **`shape`**: Button shape

### Interaction
- **`onTap`**: Called when tapped
- **`onLongPress`**: Called when long-pressed
- **`visible`**: Show/hide this child (default: `true`)

## Expansion Directions

```dart
enum SpeedDialDirection {
  up,    // Expands upward (default)
  down,  // Expands downward
  left,  // Expands to the left
  right, // Expands to the right
}
```

## Integration Patterns

### Conditional Visibility
```dart
class _HomeScreenState extends State<HomeScreen> {
  bool _showSpeedDial = true;
  
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: NotificationListener<ScrollNotification>(
        onNotification: (notification) {
          // Hide FAB when scrolling down
          setState(() {
            _showSpeedDial = notification is ScrollStartNotification ||
                             notification.metrics.pixels <= 100;
          });
          return false;
        },
        child: ListView(...),
      ),
      floatingActionButton: SpeedDial(
        visible: _showSpeedDial,
        children: [...],
      ),
    );
  }
}
```

### Programmatic Control
```dart
class _MyScreenState extends State<MyScreen> {
  ValueNotifier<bool> isDialOpen = ValueNotifier(false);
  
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: Column(
        children: [
          ElevatedButton(
            onPressed: () => isDialOpen.value = true,
            child: Text('Open Menu'),
          ),
        ],
      ),
      floatingActionButton: SpeedDial(
        openCloseDial: isDialOpen,
        onOpen: () => isDialOpen.value = true,
        onClose: () => isDialOpen.value = false,
        children: [...],
      ),
    );
  }
}
```

### Dynamic Children
```dart
class _DynamicDialState extends State<DynamicDialScreen> {
  List<String> actions = ['Edit', 'Share', 'Delete'];
  
  List<SpeedDialChild> _buildChildren() {
    return actions.map((action) {
      return SpeedDialChild(
        child: _getIconForAction(action),
        label: action,
        onTap: () => _handleAction(action),
      );
    }).toList();
  }
  
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      floatingActionButton: SpeedDial(
        children: _buildChildren(),
      ),
    );
  }
}
```

### Themed Speed Dial
```dart
SpeedDial(
  backgroundColor: Theme.of(context).primaryColor,
  foregroundColor: Theme.of(context).colorScheme.onPrimary,
  overlayColor: Theme.of(context).colorScheme.surface,
  children: [
    SpeedDialChild(
      child: Icon(Icons.edit),
      backgroundColor: Theme.of(context).colorScheme.secondary,
      labelStyle: Theme.of(context).textTheme.labelLarge,
      label: 'Edit',
    ),
  ],
)
```

### With Hero Animation
```dart
// In ListView screen
SpeedDial(
  heroTag: 'list-speed-dial',
  children: [...],
)

// In Detail screen
SpeedDial(
  heroTag: 'detail-speed-dial', // Different tag to avoid conflicts
  children: [...],
)
```

## Best Practices

### 1. Limit Number of Children
```dart
// ✅ Good: 3-6 actions
SpeedDial(
  children: [
    SpeedDialChild(...), // Action 1
    SpeedDialChild(...), // Action 2
    SpeedDialChild(...), // Action 3
  ],
)

// ❌ Bad: Too many actions (consider a drawer/menu instead)
SpeedDial(
  children: List.generate(15, (i) => SpeedDialChild(...)),
)
```

### 2. Provide Clear Labels
```dart
// ✅ Good: Descriptive labels
SpeedDialChild(
  child: Icon(Icons.photo_camera),
  label: 'Take Photo',
  onTap: _takePhoto,
)

// ❌ Bad: No label or unclear
SpeedDialChild(
  child: Icon(Icons.camera),
  // No label - user might not understand
)
```

### 3. Use Appropriate Icons
```dart
// Use standard Material icons for familiarity
SpeedDialChild(
  child: Icon(Icons.share),    // ✅ Recognizable
  label: 'Share',
)

SpeedDialChild(
  child: Icon(Icons.settings),  // ✅ Clear meaning
  label: 'Settings',
)
```

### 4. Handle State Properly
```dart
class _MyScreenState extends State<MyScreen> {
  @override
  void dispose() {
    // Clean up if using ValueNotifier for control
    _dialOpenNotifier.dispose();
    super.dispose();
  }
}
```

## Common Patterns

### Post/Create Actions
```dart
SpeedDial(
  icon: Icons.add,
  children: [
    SpeedDialChild(
      child: Icon(Icons.create),
      label: 'New Post',
      onTap: () => Navigator.pushNamed(context, '/create-post'),
    ),
    SpeedDialChild(
      child: Icon(Icons.event),
      label: 'New Event',
      onTap: () => Navigator.pushNamed(context, '/create-event'),
    ),
    SpeedDialChild(
      child: Icon(Icons.group),
      label: 'New Group',
      onTap: () => Navigator.pushNamed(context, '/create-group'),
    ),
  ],
)
```

### Edit/Delete Actions
```dart
SpeedDial(
  icon: Icons.more_vert,
  children: [
    SpeedDialChild(
      child: Icon(Icons.edit),
      backgroundColor: Colors.blue,
      label: 'Edit',
      onTap: _onEdit,
    ),
    SpeedDialChild(
      child: Icon(Icons.share),
      backgroundColor: Colors.green,
      label: 'Share',
      onTap: _onShare,
    ),
    SpeedDialChild(
      child: Icon(Icons.delete),
      backgroundColor: Colors.red,
      label: 'Delete',
      onTap: _onDelete,
    ),
  ],
)
```

### Media Selection
```dart
SpeedDial(
  animatedIcon: AnimatedIcons.add_event,
  children: [
    SpeedDialChild(
      child: Icon(Icons.camera_alt),
      label: 'Camera',
      onTap: () => _pickImage(ImageSource.camera),
    ),
    SpeedDialChild(
      child: Icon(Icons.photo_library),
      label: 'Gallery',
      onTap: () => _pickImage(ImageSource.gallery),
    ),
    SpeedDialChild(
      child: Icon(Icons.video_library),
      label: 'Video',
      onTap: () => _pickVideo(),
    ),
  ],
)
```

## Styling Examples

### Minimal Design
```dart
SpeedDial(
  icon: Icons.add,
  backgroundColor: Colors.white,
  foregroundColor: Colors.black,
  overlayOpacity: 0.3,
  elevation: 2.0,
  children: [
    SpeedDialChild(
      child: Icon(Icons.edit),
      backgroundColor: Colors.white,
      foregroundColor: Colors.black,
      elevation: 2.0,
    ),
  ],
)
```

### Bold & Colorful
```dart
SpeedDial(
  icon: Icons.menu,
  backgroundColor: Colors.purple,
  foregroundColor: Colors.white,
  overlayColor: Colors.purple,
  overlayOpacity: 0.7,
  elevation: 12.0,
  children: [
    SpeedDialChild(
      child: Icon(Icons.star),
      backgroundColor: Colors.amber,
      label: 'Favorite',
      labelBackgroundColor: Colors.amber.shade100,
    ),
    SpeedDialChild(
      child: Icon(Icons.bookmark),
      backgroundColor: Colors.green,
      label: 'Save',
      labelBackgroundColor: Colors.green.shade100,
    ),
  ],
)
```

## Common Issues & Solutions

### 1. Multiple FABs Conflict
```dart
// Use unique heroTag for each SpeedDial
Scaffold(
  floatingActionButton: SpeedDial(
    heroTag: 'unique-tag-1', // Required
    children: [...],
  ),
)
```

### 2. Overlay Blocking UI
```dart
// Reduce overlay opacity or disable
SpeedDial(
  overlayOpacity: 0.2, // Lower opacity
  // or
  renderOverlay: false, // No overlay
  children: [...],
)
```

### 3. Children Not Showing
- Check if `visible` property is `true`
- Verify children list is not empty
- Ensure sufficient space for expansion

### 4. Animation Stuttering
```dart
// Use simpler curve
SpeedDial(
  curve: Curves.easeInOut, // Instead of bounceIn
  children: [...],
)
```

## Accessibility

### Screen Reader Support
```dart
SpeedDial(
  tooltip: 'Quick Actions Menu', // Read by screen readers
  children: [
    SpeedDialChild(
      child: Icon(Icons.edit, semanticLabel: 'Edit item'),
      label: 'Edit', // Also used by screen readers
    ),
  ],
)
```

## Performance Considerations

- **Limit Children**: Keep to 3-6 actions for best UX
- **Avoid Heavy Widgets**: Use simple Icons in children
- **Dispose Properly**: Clean up ValueNotifiers and controllers

## Documentation Requirements
- Document all customization options
- Include visual examples for common patterns
- Note performance implications of many children
- Specify Material Design guidelines

Example:
```dart
/// A Material Design Speed Dial (expandable FAB menu).
///
/// Displays a floating action button that expands to reveal multiple
/// action buttons when tapped.
///
/// [children] The list of action buttons shown when expanded.
/// Required and must not be empty.
///
/// [icon] Icon shown when menu is closed. Defaults to [Icons.add].
///
/// [activeIcon] Icon shown when menu is open. Defaults to [icon].
///
/// [direction] Direction of expansion. Defaults to [SpeedDialDirection.up].
///
/// Best practices:
/// - Limit to 3-6 children for optimal UX
/// - Provide clear, concise labels
/// - Use recognizable icons
/// - Set unique [heroTag] if multiple FABs exist
///
/// Example:
/// ```dart
/// SpeedDial(
///   icon: Icons.add,
///   children: [
///     SpeedDialChild(
///       child: Icon(Icons.photo),
///       label: 'Add Photo',
///       onTap: () => _addPhoto(),
///     ),
///   ],
/// );
/// ```
class SpeedDial extends StatefulWidget { ... }
```

## Reference

### Key Files
- `lib/src/speed_dial.dart`: Main widget
- `lib/src/speed_dial_child.dart`: Child button widget
- `lib/src/speed_dial_direction.dart`: Direction enum

### Dependencies
- Pure Flutter (no external dependencies)

### Related Documentation
- Material Design FAB: https://m3.material.io/components/floating-action-button
- Flutter FAB: https://api.flutter.dev/flutter/material/FloatingActionButton-class.html
- Flutter AnimatedIcons: https://api.flutter.dev/flutter/material/AnimatedIcons-class.html
