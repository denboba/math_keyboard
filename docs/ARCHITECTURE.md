# Math Keyboard Architecture Overview

## System Architecture

### High-Level Component Diagram

```
┌─────────────────────────────────────────────────────────────┐
│                     Application Layer                        │
│  ┌─────────────┐  ┌─────────────┐  ┌──────────────────┐   │
│  │Quiz App     │  │Note Editor  │  │ Learning Module  │   │
│  └─────────────┘  └─────────────┘  └──────────────────┘   │
└─────────────────────────────────────────────────────────────┘
                             │
                             ▼
┌─────────────────────────────────────────────────────────────┐
│                 Math Keyboard Package API                    │
│  ┌──────────────┐  ┌──────────────────┐  ┌──────────────┐  │
│  │  MathField   │  │  MathFormField   │  │MathKeyboard  │  │
│  └──────────────┘  └──────────────────┘  └──────────────┘  │
│                             │                                │
│  ┌─────────────────────────────────────────────────────┐   │
│  │     MathFieldEditingController                       │   │
│  │  - Manages expression tree (TeXNode)                 │   │
│  │  - Handles cursor position                           │   │
│  │  - Processes input events                            │   │
│  │  - Generates LaTeX output                            │   │
│  └─────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────┘
                             │
                ┌────────────┴────────────┐
                ▼                         ▼
┌──────────────────────────┐  ┌──────────────────────────┐
│  Expression Tree Layer   │  │   Rendering Layer        │
│  ┌────────────────────┐  │  │  ┌────────────────────┐  │
│  │ TeXNode Tree       │  │  │  │ flutter_math_fork  │  │
│  │ - TextNode         │  │  │  │ - TeX Renderer     │  │
│  │ - FunctionNode     │  │  │  │ - Math Typesetting │  │
│  │ - FractionNode     │  │  │  └────────────────────┘  │
│  │ - PowerNode        │  │  │           │               │
│  │ - MatrixNode       │  │  │           ▼               │
│  │ - CursorNode       │  │  │  ┌────────────────────┐  │
│  └────────────────────┘  │  │  │  Flutter Widget    │  │
│           │               │  │  │  Tree              │  │
│           ▼               │  │  └────────────────────┘  │
│  ┌────────────────────┐  │  └──────────────────────────┘
│  │ LaTeX Generator    │  │
│  │ - buildTeXString() │  │
│  └────────────────────┘  │
└──────────────────────────┘
           │
           ▼
┌──────────────────────────┐
│  Math Expression Layer   │
│  ┌────────────────────┐  │
│  │ TeXParser          │  │
│  │ (tex2math)         │  │
│  │                    │  │
│  │ String → Expr      │  │
│  └────────────────────┘  │
│  ┌────────────────────┐  │
│  │ Math2TeX           │  │
│  │ Expr → TeXNode     │  │
│  └────────────────────┘  │
│           │               │
│           ▼               │
│  ┌────────────────────┐  │
│  │ math_expressions   │  │
│  │ - Evaluation       │  │
│  │ - Simplification   │  │
│  └────────────────────┘  │
└──────────────────────────┘
```

## Data Flow

### Input Event Flow

```
User Action (Touch/Keyboard)
         │
         ▼
┌──────────────────────┐
│  MathField Widget    │
│  - GestureDetector   │
│  - RawKeyboardListener
└──────────────────────┘
         │
         ▼
┌─────────────────────────────┐
│ MathFieldEditingController  │
│ - _handleInput()            │
└─────────────────────────────┘
         │
         ├─────────────┬──────────────┬────────────┐
         ▼             ▼              ▼            ▼
    [Number]    [Operator]    [Function]    [Navigation]
         │             │              │            │
         ▼             ▼              ▼            ▼
   addTeX()      addTeX()    insertStructure() navigateCursor()
         │             │              │            │
         └─────────────┴──────────────┴────────────┘
                       │
                       ▼
           ┌────────────────────┐
           │  Update TeXNode    │
           │  Tree              │
           └────────────────────┘
                       │
                       ▼
           ┌────────────────────┐
           │  buildTeXString()  │
           └────────────────────┘
                       │
                       ▼
           ┌────────────────────┐
           │  notifyListeners() │
           └────────────────────┘
                       │
                       ▼
           ┌────────────────────┐
           │  Widget Rebuild    │
           └────────────────────┘
                       │
                       ▼
           ┌────────────────────┐
           │  flutter_math_fork │
           │  Renders LaTeX     │
           └────────────────────┘
```

### Expression Evaluation Flow

```
User Submits Expression
         │
         ▼
┌──────────────────────┐
│  MathField           │
│  onSubmitted()       │
└──────────────────────┘
         │
         ▼
┌──────────────────────┐
│  Get LaTeX String    │
│  controller.tex      │
└──────────────────────┘
         │
         ▼
┌──────────────────────┐
│  TeXParser           │
│  parse(latex)        │
└──────────────────────┘
         │
         ▼
┌──────────────────────┐
│  math_expressions    │
│  Expression object   │
└──────────────────────┘
         │
         ├──────────────┬──────────────┐
         ▼              ▼              ▼
    evaluate()    simplify()    toString()
         │              │              │
         ▼              ▼              ▼
    [Result]      [Simplified]   [Display]
```

## Core Classes

### MathFieldEditingController

**Responsibilities:**
- Manage the expression tree (TeXNode root)
- Track cursor position
- Handle input events (characters, functions, navigation)
- Generate LaTeX output
- Notify listeners on changes
- Support undo/redo

**Key Methods:**
```dart
class MathFieldEditingController extends ValueNotifier<String> {
  // State
  TeXNode _root;
  List<TeXNode> _history;
  int _historyIndex;
  
  // Public API
  String get tex;
  void clear();
  void updateExpression(String latex);
  
  // Input handling
  void _insertCharacter(String char);
  void _insertFunction(String function, List<TeXArg> args);
  void _navigateCursor(Direction direction);
  void _deleteAtCursor();
  
  // History
  void undo();
  void redo();
  bool get canUndo;
  bool get canRedo;
}
```

### TeXNode

**Responsibilities:**
- Represent a node in the expression tree
- Manage children nodes
- Track cursor position within node
- Navigate between nodes
- Generate LaTeX for this node and its children

**Key Methods:**
```dart
class TeXNode {
  // State
  TeXFunction? parent;
  List<TeX> children;
  int cursorPosition;
  
  // Cursor management
  void setCursor();
  void removeCursor();
  bool cursorAtTheEnd();
  
  // Navigation
  NavigationState shiftCursorLeft();
  NavigationState shiftCursorRight();
  
  // Editing
  void addTeX(TeX tex);
  NavigationState remove();
  
  // Output
  String buildTeXString({
    Color? cursorColor,
    bool placeholderWhenEmpty = true,
  });
}
```

### TeXFunction

**Responsibilities:**
- Represent mathematical functions (fractions, powers, trig functions, etc.)
- Manage function-specific arguments
- Generate function-specific LaTeX

**Structure:**
```dart
class TeXFunction extends TeX {
  String name;              // e.g., "frac", "sin", "sqrt"
  List<TeXArg> args;        // Argument types
  List<TeXNode> children;   // One TeXNode per argument
  
  String buildTeXString();
}

enum TeXArg {
  braces,      // {content}
  brackets,    // [content]
  parentheses  // (content)
}
```

### MathField Widget

**Responsibilities:**
- Display the math expression
- Handle focus and keyboard input
- Show/hide the math keyboard
- Integrate with Flutter's InputDecoration system

**Structure:**
```dart
class MathField extends StatefulWidget {
  final MathFieldEditingController? controller;
  final FocusNode? focusNode;
  final InputDecoration? decoration;
  final MathKeyboardType keyboardType;
  final List<String> variables;
  final ValueChanged<String>? onChanged;
  final ValueChanged<String>? onSubmitted;
  final bool autofocus;
}
```

### MathKeyboard Widget

**Responsibilities:**
- Display the on-screen keyboard
- Handle button presses
- Manage keyboard tabs/pages
- Report keyboard size for view insets
- Support long-press menus

**Structure:**
```dart
class MathKeyboard extends StatelessWidget {
  final MathFieldEditingController controller;
  final MathKeyboardType type;
  final List<String> variables;
  final VoidCallback? onSubmit;
  final MathKeyboardViewInsetsState? insetsState;
  final Animation<double>? slideAnimation;
}
```

## State Management Patterns

### Controller Pattern (Current Implementation)

```
┌──────────────────────────────┐
│  Widget Tree                 │
│  ┌────────────────────────┐  │
│  │  MathField             │  │
│  │  - controller          │◄─┼─ Reference
│  └────────────────────────┘  │
│                              │
│  ┌────────────────────────┐  │
│  │  MathKeyboard          │  │
│  │  - controller          │◄─┼─ Same Reference
│  └────────────────────────┘  │
└──────────────────────────────┘
         │
         ▼
┌──────────────────────────────┐
│  MathFieldEditingController  │
│  extends ValueNotifier        │
│                              │
│  - Manages state             │
│  - Notifies listeners        │
│  - Lifecycle managed by      │
│    parent widget             │
└──────────────────────────────┘
```

### Alternative: Provider Pattern

```
┌──────────────────────────────┐
│  ChangeNotifierProvider      │
│  ┌────────────────────────┐  │
│  │  MathExpression        │  │
│  │  Model                 │  │
│  └────────────────────────┘  │
└──────────────────────────────┘
         │
         ├─────────────┬────────────┐
         ▼             ▼            ▼
   [MathField]   [MathKeyboard]  [Controls]
         │             │            │
         └─────────────┴────────────┘
                   │
                   ▼
            Consumer/Selector
```

## Memory Management

### Node Lifecycle

```
Creation:
- User presses button → new node allocated
- Node added to parent's children list
- Parent reference set

Modification:
- Direct mutation (efficient)
- Cursor moves → node references updated
- No new allocations for navigation

Deletion:
- Node removed from parent's children
- References cleared
- Garbage collected by Dart VM

History:
- Deep copy for undo/redo
- Limited to last N states (default: 100)
- Oldest states removed when limit exceeded
```

### Widget Rebuild Optimization

```dart
class _MathFieldState extends State<MathField> {
  @override
  Widget build(BuildContext context) {
    return RepaintBoundary(  // Isolate repaints
      child: ValueListenableBuilder<String>(
        valueListenable: _controller,
        builder: (context, value, child) {
          // Only rebuilds when controller notifies
          return Math.tex(value);
        },
      ),
    );
  }
}
```

## Performance Considerations

### Rendering Optimization

1. **RepaintBoundary:** Wrap math display to prevent parent repaints
2. **Caching:** Cache rendered Math widgets when possible
3. **Lazy Loading:** Only render visible keyboard buttons
4. **Debouncing:** Delay expensive operations (e.g., expression evaluation)

### Navigation Optimization

1. **Linked Structure:** Nodes reference parent for O(1) traversal
2. **Cursor Cache:** Store cursor path for quick access
3. **Incremental Updates:** Only update changed portions of LaTeX string

### Memory Optimization

1. **Node Pooling:** Reuse deleted nodes (optional)
2. **History Limits:** Cap undo/redo stack size
3. **Weak References:** Use weak refs for non-essential caches

## Error Handling

### Input Validation

```dart
void _insertCharacter(String char) {
  try {
    // Validate character
    if (!_isValidCharacter(char)) {
      _showError('Invalid character: $char');
      return;
    }
    
    // Insert into tree
    _root.addTeX(TextNode(char));
    notifyListeners();
  } catch (e) {
    _handleError(e);
  }
}
```

### LaTeX Generation

```dart
String buildTeXString() {
  try {
    return _root.buildTeXString();
  } catch (e) {
    debugPrint('Error generating LaTeX: $e');
    return '\\text{Error}';  // Fallback
  }
}
```

### Expression Parsing

```dart
Expression? parse(String latex) {
  try {
    return TeXParser(latex).parse();
  } on FormatException catch (e) {
    _showError('Invalid expression: ${e.message}');
    return null;
  } catch (e) {
    _showError('Parsing error: $e');
    return null;
  }
}
```

## Testing Strategy

### Unit Tests
- TeXNode operations (add, remove, navigate)
- LaTeX generation correctness
- tex2math and math2tex conversions
- Controller state management

### Widget Tests
- MathField rendering
- Button presses trigger correct actions
- Focus management
- Keyboard show/hide animations

### Integration Tests
- Complete user flows (input → display → evaluate)
- Multi-field scenarios
- Undo/redo workflows
- Physical keyboard input

### Golden Tests
- Visual regression testing for rendered math
- Consistent appearance across platforms

## Security Considerations

### Input Sanitization
- Validate all user input before insertion
- Prevent LaTeX injection attacks
- Limit expression depth to prevent stack overflow

### Resource Limits
- Max expression size (characters, nodes)
- Rendering timeouts
- Memory usage caps

---

**Last Updated:** December 2025  
**Version:** 1.0
