# Math Keyboard API Reference

Complete API documentation for the math_keyboard package.

## Table of Contents

1. [Widgets](#widgets)
2. [Controllers](#controllers)
3. [Enums](#enums)
4. [Conversion Utilities](#conversion-utilities)
5. [Configuration](#configuration)

---

## Widgets

### MathField

Primary widget for displaying and editing math expressions.

```dart
class MathField extends StatefulWidget {
  const MathField({
    Key? key,
    this.controller,
    this.focusNode,
    this.decoration,
    this.keyboardType = MathKeyboardType.expression,
    this.variables = const [],
    this.onChanged,
    this.onSubmitted,
    this.autofocus = false,
  });
}
```

#### Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `controller` | `MathFieldEditingController?` | `null` | Controller for managing field content. If null, creates internal controller. |
| `focusNode` | `FocusNode?` | `null` | Focus node for managing focus. If null, creates internal focus node. |
| `decoration` | `InputDecoration?` | `null` | Decoration for the input field (same as TextField). |
| `keyboardType` | `MathKeyboardType` | `expression` | Type of keyboard to display. |
| `variables` | `List<String>` | `[]` | Allowed variables in expression mode. |
| `onChanged` | `ValueChanged<String>?` | `null` | Callback when content changes. Receives LaTeX string. |
| `onSubmitted` | `ValueChanged<String>?` | `null` | Callback when user submits (presses enter). Receives LaTeX string. |
| `autofocus` | `bool` | `false` | Whether to auto-focus on mount. |

#### Example

```dart
MathField(
  controller: _controller,
  keyboardType: MathKeyboardType.expression,
  variables: ['x', 'y', 'z'],
  decoration: InputDecoration(
    labelText: 'Enter expression',
    border: OutlineInputBorder(),
  ),
  onChanged: (latex) => print('Changed: $latex'),
  onSubmitted: (latex) => print('Submitted: $latex'),
  autofocus: true,
)
```

---

### MathFormField

Form field variant with validation support.

```dart
class MathFormField extends FormField<String> {
  const MathFormField({
    Key? key,
    this.controller,
    this.focusNode,
    this.decoration,
    this.keyboardType = MathKeyboardType.expression,
    this.variables = const [],
    this.validator,
    this.onChanged,
    this.onSubmitted,
    this.autofocus = false,
    this.initialValue,
  });
}
```

#### Additional Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `validator` | `FormFieldValidator<String>?` | `null` | Validation function. Returns error string or null. |
| `initialValue` | `String?` | `null` | Initial LaTeX value. Cannot be used with controller. |

#### Example

```dart
MathFormField(
  decoration: InputDecoration(
    labelText: 'Enter f(x)',
  ),
  validator: (value) {
    if (value == null || value.isEmpty) {
      return 'Please enter an expression';
    }
    return null;
  },
  onSubmitted: (latex) => _submitForm(),
)
```

---

### MathKeyboard

The on-screen mathematical keyboard widget.

```dart
class MathKeyboard extends StatelessWidget {
  const MathKeyboard({
    Key? key,
    required this.controller,
    this.type = MathKeyboardType.expression,
    this.variables = const [],
    this.onSubmit,
    this.insetsState,
    this.slideAnimation,
    this.padding = const EdgeInsets.only(bottom: 4, left: 4, right: 4),
  });
}
```

#### Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `controller` | `MathFieldEditingController` | Yes | Controller to update when buttons are pressed. |
| `type` | `MathKeyboardType` | No | Type of keyboard layout. |
| `variables` | `List<String>` | No | Available variables to show. |
| `onSubmit` | `VoidCallback?` | No | Callback when submit button is pressed. |
| `insetsState` | `MathKeyboardViewInsetsState?` | No | State for reporting keyboard size. |
| `slideAnimation` | `Animation<double>?` | No | Animation for keyboard show/hide. |
| `padding` | `EdgeInsets` | Default padding | Padding around keyboard. |

#### Example

```dart
// Usually you don't create this directly - it's shown automatically by MathField
// But you can if needed:
MathKeyboard(
  controller: _controller,
  type: MathKeyboardType.expression,
  variables: ['x', 'y'],
  onSubmit: () => print('Submitted'),
)
```

---

### MathKeyboardViewInsets

Widget that reports math keyboard size as view insets (like system keyboard).

```dart
class MathKeyboardViewInsets extends StatefulWidget {
  const MathKeyboardViewInsets({
    Key? key,
    required this.child,
  });
  
  final Widget child;
}
```

#### Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `child` | `Widget` | Yes | Child widget (typically Scaffold). |

#### Example

```dart
MathKeyboardViewInsets(
  child: Scaffold(
    appBar: AppBar(title: Text('Math Input')),
    body: Column(
      children: [
        MathField(/* ... */),
      ],
    ),
  ),
)
```

#### Helper Methods

```dart
// Check if math keyboard is showing
bool showing = MathKeyboardViewInsetsQuery.mathKeyboardShowingIn(context);

// Check if any keyboard is showing (system or math)
bool anyKeyboard = MathKeyboardViewInsetsQuery.keyboardShowingIn(context);
```

---

## Controllers

### MathFieldEditingController

Controller for managing math field content and cursor position.

```dart
class MathFieldEditingController extends ValueNotifier<String> {
  MathFieldEditingController();
}
```

#### Properties

| Property | Type | Description |
|----------|------|-------------|
| `tex` | `String` | Current LaTeX representation (read-only). |
| `currentEditingValue` | `String` | Current editing value. |

#### Methods

| Method | Returns | Description |
|--------|---------|-------------|
| `clear()` | `void` | Clears all content. |
| `updateExpression(String latex)` | `void` | Sets content to the given LaTeX string. |

#### Example

```dart
final controller = MathFieldEditingController();

// Set initial value
controller.updateExpression(r'x^2 + 1');

// Get current value
print(controller.tex);  // "x^2 + 1"

// Clear
controller.clear();

// Listen to changes
controller.addListener(() {
  print('Changed to: ${controller.tex}');
});

// Dispose when done
controller.dispose();
```

---

## Enums

### MathKeyboardType

Defines the type of keyboard to display.

```dart
enum MathKeyboardType {
  /// Full expression keyboard with functions and operators
  expression,
  
  /// Number-only keyboard (0-9, decimal, +/-)
  numberOnly,
}
```

#### Usage

```dart
MathField(
  keyboardType: MathKeyboardType.expression,  // Full math keyboard
)

MathField(
  keyboardType: MathKeyboardType.numberOnly,  // Numbers only
)
```

---

## Conversion Utilities

### TeXParser

Parses LaTeX strings into evaluable math expressions.

```dart
class TeXParser {
  TeXParser(this.texString);
  
  final String texString;
  
  /// Parse the LaTeX string into a math expression
  Expression parse();
}
```

#### Example

```dart
import 'package:math_keyboard/math_keyboard.dart';
import 'package:math_expressions/math_expressions.dart';

// Parse LaTeX to expression
final parser = TeXParser(r'x^2 + 2*x + 1');
final expression = parser.parse();

// Evaluate
final context = ContextModel();
context.bindVariable(Variable('x'), Number(5));
final result = expression.evaluate(EvaluationType.REAL, context);
print(result);  // 36.0
```

---

### convertMathExpressionToTeXNode

Converts a math expression to an internal TeXNode representation.

```dart
TeXNode convertMathExpressionToTeXNode(Expression expression);
```

#### Example

```dart
import 'package:math_expressions/math_expressions.dart';
import 'package:math_keyboard/math_keyboard.dart';

// Create expression
final x = Variable('x');
final expr = x * x + Number(1);

// Convert to TeXNode
final texNode = convertMathExpressionToTeXNode(expr);

// Get LaTeX string
final latex = texNode.buildTeXString();
print(latex);  // "x^2 + 1" or similar
```

---

## Configuration

### Decimal Separator

The keyboard automatically adapts decimal separator based on locale.

```dart
// Use system locale (default)
MathField(/* ... */)

// Override locale
Localizations.override(
  context: context,
  locale: Locale('de', 'DE'),  // German (comma separator)
  child: MathField(/* ... */),
)
```

**Behavior:**
- Locales with comma separator (de, fr, es, etc.): Shows `,` on keyboard
- Other locales: Shows `.` on keyboard
- Physical keyboard: Accepts both `.` and `,` regardless of locale

---

## Internal Types (Advanced)

### TeXNode

Internal representation of an expression node.

```dart
class TeXNode {
  TeXNode(this.parent);
  
  TeXFunction? parent;
  int cursorPosition;
  List<TeX> children;
  
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

Represents mathematical functions with arguments.

```dart
class TeXFunction extends TeX {
  String name;
  List<TeXArg> args;
  List<TeXNode> children;
  
  String buildTeXString();
}
```

### TeXArg

Argument types for functions.

```dart
enum TeXArg {
  braces,      // {argument}
  brackets,    // [argument]
  parentheses, // (argument)
}
```

### NavigationState

Result of cursor navigation operations.

```dart
enum NavigationState {
  success,  // Navigation succeeded
  end,      // At boundary, cannot navigate further
  func,     // Entered/exited a function
}
```

---

## LaTeX Output Reference

### Supported LaTeX Commands

#### Numbers and Variables
- Digits: `0`, `1`, `2`, ..., `9`
- Decimal: `.` or `,` (locale-dependent)
- Variables: `x`, `y`, `z`, custom variables

#### Basic Operators
- Addition: `+`
- Subtraction: `-`
- Multiplication: `\times` or `\cdot`
- Division: `\div` or `/`
- Equals: `=`
- Inequalities: `<`, `>`, `\leq`, `\geq`, `\neq`

#### Structure Commands
- Fraction: `\frac{numerator}{denominator}`
- Power: `base^{exponent}`
- Subscript: `base_{subscript}`
- Square root: `\sqrt{radicand}`
- Nth root: `\sqrt[n]{radicand}`
- Absolute value: `|expression|`

#### Functions
- Trigonometry: `\sin`, `\cos`, `\tan`, `\csc`, `\sec`, `\cot`
- Inverse trig: `\sin^{-1}`, `\cos^{-1}`, `\tan^{-1}`
- Logarithms: `\ln`, `\log`
- Exponential: `e^{x}`

#### Calculus
- Integral: `\int expression \, dx`
- Definite integral: `\int_{a}^{b} expression \, dx`
- Derivative: `\frac{d}{dx}`, `\frac{\partial}{\partial x}`
- Limit: `\lim_{x \to a} expression`
- Summation: `\sum_{i=a}^{b} expression`
- Product: `\prod_{i=a}^{b} expression`

#### Greek Letters
- Lowercase: `\alpha`, `\beta`, `\gamma`, `\delta`, `\epsilon`, `\theta`, `\lambda`, `\mu`, `\pi`, `\sigma`, `\omega`
- Uppercase: `\Gamma`, `\Delta`, `\Theta`, `\Lambda`, `\Sigma`, `\Pi`, `\Omega`

#### Special Symbols
- Infinity: `\infty`
- Pi: `\pi`
- Euler's number: `e`
- Imaginary unit: `i`

#### Grouping
- Parentheses: `(`, `)`
- Brackets: `[`, `]`
- Braces: `\{`, `\}`
- Angle brackets: `\langle`, `\rangle`

---

## Error Handling

### Common Exceptions

#### FormatException
Thrown when LaTeX parsing fails.

```dart
try {
  final expr = TeXParser(invalidLatex).parse();
} on FormatException catch (e) {
  print('Parse error: ${e.message}');
}
```

#### ArgumentError
Thrown when invalid arguments are provided.

```dart
try {
  controller.updateExpression('');
} on ArgumentError catch (e) {
  print('Argument error: ${e.message}');
}
```

### Best Practices

1. **Always validate user input** before parsing
2. **Wrap parse operations** in try-catch blocks
3. **Provide user-friendly error messages**
4. **Log errors** for debugging
5. **Have fallback behavior** for parse failures

---

## Migration Guide

### From Version 0.2.x to 0.3.x

No breaking changes. New features:
- Improved physical keyboard support
- Better cursor navigation
- Additional math functions

### Future Versions

Check [CHANGELOG.md](../CHANGELOG.md) for version-specific changes.

---

## Examples

### Complete Example App

```dart
import 'package:flutter/material.dart';
import 'package:math_keyboard/math_keyboard.dart';
import 'package:math_expressions/math_expressions.dart';

void main() {
  runApp(MyApp());
}

class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Math Keyboard Demo',
      theme: ThemeData(primarySwatch: Colors.blue),
      home: MathInputPage(),
    );
  }
}

class MathInputPage extends StatefulWidget {
  @override
  _MathInputPageState createState() => _MathInputPageState();
}

class _MathInputPageState extends State<MathInputPage> {
  late final MathFieldEditingController _controller;
  String _result = '';

  @override
  void initState() {
    super.initState();
    _controller = MathFieldEditingController();
  }

  @override
  void dispose() {
    _controller.dispose();
    super.dispose();
  }

  void _evaluate() {
    try {
      final expr = TeXParser(_controller.tex).parse();
      final context = ContextModel();
      context.bindVariable(Variable('x'), Number(10));
      
      final result = expr.evaluate(EvaluationType.REAL, context);
      setState(() {
        _result = 'Result: $result';
      });
    } catch (e) {
      setState(() {
        _result = 'Error: $e';
      });
    }
  }

  @override
  Widget build(BuildContext context) {
    return MathKeyboardViewInsets(
      child: Scaffold(
        appBar: AppBar(
          title: Text('Math Input'),
        ),
        body: Padding(
          padding: EdgeInsets.all(16),
          child: Column(
            children: [
              MathField(
                controller: _controller,
                keyboardType: MathKeyboardType.expression,
                variables: ['x', 'y', 'z'],
                decoration: InputDecoration(
                  labelText: 'Enter expression',
                  border: OutlineInputBorder(),
                ),
                onSubmitted: (_) => _evaluate(),
              ),
              SizedBox(height: 16),
              ElevatedButton(
                onPressed: _evaluate,
                child: Text('Evaluate at x=10'),
              ),
              SizedBox(height: 16),
              Text(
                _result,
                style: TextStyle(fontSize: 18),
              ),
            ],
          ),
        ),
      ),
    );
  }
}
```

---

## Support

- **Issues:** https://github.com/simpleclub/math_keyboard/issues
- **Discussions:** https://github.com/simpleclub/math_keyboard/discussions
- **Documentation:** https://pub.dev/documentation/math_keyboard

---

**Last Updated:** December 2025  
**Package Version:** 0.3.3
