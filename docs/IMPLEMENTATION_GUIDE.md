# Math Keyboard Implementation Guide

This guide provides practical steps for implementing the math keyboard specification in your Flutter application.

## Table of Contents

1. [Getting Started](#getting-started)
2. [Basic Implementation](#basic-implementation)
3. [Advanced Features](#advanced-features)
4. [Common Patterns](#common-patterns)
5. [Troubleshooting](#troubleshooting)

---

## Getting Started

### Prerequisites

- Flutter SDK >= 3.22.2
- Dart SDK >= 3.3.0

### Installation

Add to your `pubspec.yaml`:

```yaml
dependencies:
  math_keyboard: ^0.3.3
  flutter_math_fork: ^0.7.3  # For LaTeX rendering
  math_expressions: ^2.5.0   # For evaluation (optional)
```

Run:
```bash
flutter pub get
```

### Import

```dart
import 'package:math_keyboard/math_keyboard.dart';
```

---

## Basic Implementation

### 1. Simple Math Field

The most basic usage - just add a MathField widget:

```dart
import 'package:flutter/material.dart';
import 'package:math_keyboard/math_keyboard.dart';

class SimpleMathInput extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text('Math Input')),
      body: MathKeyboardViewInsets(
        child: Center(
          child: Padding(
            padding: EdgeInsets.all(16),
            child: MathField(
              keyboardType: MathKeyboardType.expression,
              decoration: InputDecoration(
                labelText: 'Enter expression',
                border: OutlineInputBorder(),
              ),
            ),
          ),
        ),
      ),
    );
  }
}
```

**Key Points:**
- Wrap your Scaffold with `MathKeyboardViewInsets` for proper keyboard behavior
- `MathKeyboardType.expression` shows the full math keyboard
- Use `MathKeyboardType.numberOnly` for numeric input only

### 2. With Controller

Manage the input programmatically:

```dart
class ControlledMathInput extends StatefulWidget {
  @override
  _ControlledMathInputState createState() => _ControlledMathInputState();
}

class _ControlledMathInputState extends State<ControlledMathInput> {
  late final MathFieldEditingController _controller;

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

  void _clearInput() {
    _controller.clear();
  }

  void _setExample() {
    _controller.updateExpression(r'x^2 + 2x + 1');
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text('Math Input with Controller'),
        actions: [
          IconButton(
            icon: Icon(Icons.clear),
            onPressed: _clearInput,
          ),
          IconButton(
            icon: Icon(Icons.lightbulb),
            onPressed: _setExample,
          ),
        ],
      ),
      body: MathKeyboardViewInsets(
        child: Padding(
          padding: EdgeInsets.all(16),
          child: MathField(
            controller: _controller,
            decoration: InputDecoration(
              labelText: 'Enter expression',
              border: OutlineInputBorder(),
            ),
          ),
        ),
      ),
    );
  }
}
```

### 3. Handling Input Changes

React to user input:

```dart
class ReactiveM athInput extends StatefulWidget {
  @override
  _ReactiveMathInputState createState() => _ReactiveMathInputState();
}

class _ReactiveMathInputState extends State<ReactiveMathInput> {
  late final MathFieldEditingController _controller;
  String _currentLatex = '';

  @override
  void initState() {
    super.initState();
    _controller = MathFieldEditingController();
    _controller.addListener(_onInputChanged);
  }

  void _onInputChanged() {
    setState(() {
      _currentLatex = _controller.tex;
    });
  }

  @override
  void dispose() {
    _controller.removeListener(_onInputChanged);
    _controller.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text('Reactive Math Input')),
      body: MathKeyboardViewInsets(
        child: Column(
          children: [
            Padding(
              padding: EdgeInsets.all(16),
              child: MathField(
                controller: _controller,
                decoration: InputDecoration(
                  labelText: 'Enter expression',
                  border: OutlineInputBorder(),
                ),
                onChanged: (String latex) {
                  print('Changed: $latex');
                },
                onSubmitted: (String latex) {
                  print('Submitted: $latex');
                  _evaluateExpression(latex);
                },
              ),
            ),
            Padding(
              padding: EdgeInsets.all(16),
              child: Text(
                'LaTeX: $_currentLatex',
                style: TextStyle(fontFamily: 'monospace'),
              ),
            ),
          ],
        ),
      ),
    );
  }

  void _evaluateExpression(String latex) {
    // Evaluation logic (see Advanced Features section)
  }
}
```

### 4. Form Integration

Use in Flutter forms with validation:

```dart
class MathForm extends StatefulWidget {
  @override
  _MathFormState createState() => _MathFormState();
}

class _MathFormState extends State<MathForm> {
  final _formKey = GlobalKey<FormState>();
  late final MathFieldEditingController _controller;

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

  String? _validateExpression(String? value) {
    if (value == null || value.isEmpty) {
      return 'Please enter an expression';
    }
    
    // Custom validation logic
    if (!value.contains('x')) {
      return 'Expression must contain variable x';
    }
    
    return null;
  }

  void _submitForm() {
    if (_formKey.currentState!.validate()) {
      print('Valid expression: ${_controller.tex}');
      // Process the expression
    }
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text('Math Form')),
      body: MathKeyboardViewInsets(
        child: Form(
          key: _formKey,
          child: Padding(
            padding: EdgeInsets.all(16),
            child: Column(
              children: [
                MathFormField(
                  controller: _controller,
                  decoration: InputDecoration(
                    labelText: 'Enter f(x)',
                    border: OutlineInputBorder(),
                  ),
                  validator: _validateExpression,
                ),
                SizedBox(height: 16),
                ElevatedButton(
                  onPressed: _submitForm,
                  child: Text('Submit'),
                ),
              ],
            ),
          ),
        ),
      ),
    );
  }
}
```

---

## Advanced Features

### 1. Expression Evaluation

Convert LaTeX to evaluable expressions:

```dart
import 'package:math_expressions/math_expressions.dart';
import 'package:math_keyboard/math_keyboard.dart';

class ExpressionEvaluator {
  /// Parse LaTeX and evaluate at a given value
  double? evaluate(String latex, {double x = 0}) {
    try {
      // Parse LaTeX to math expression
      final expr = TeXParser(latex).parse();
      
      // Create context with variable bindings
      final context = ContextModel();
      context.bindVariable(Variable('x'), Number(x));
      
      // Evaluate
      final result = expr.evaluate(EvaluationType.REAL, context);
      return result as double?;
    } catch (e) {
      print('Evaluation error: $e');
      return null;
    }
  }

  /// Plot function over a range
  List<Offset> plotFunction(String latex, {
    double xMin = -10,
    double xMax = 10,
    int points = 100,
  }) {
    final step = (xMax - xMin) / points;
    final plotPoints = <Offset>[];

    for (var i = 0; i <= points; i++) {
      final x = xMin + i * step;
      final y = evaluate(latex, x: x);
      if (y != null && y.isFinite) {
        plotPoints.add(Offset(x, y));
      }
    }

    return plotPoints;
  }
}

// Usage
class FunctionPlotter extends StatefulWidget {
  @override
  _FunctionPlotterState createState() => _FunctionPlotterState();
}

class _FunctionPlotterState extends State<FunctionPlotter> {
  late final MathFieldEditingController _controller;
  final _evaluator = ExpressionEvaluator();
  List<Offset> _plotPoints = [];

  @override
  void initState() {
    super.initState();
    _controller = MathFieldEditingController();
  }

  void _updatePlot() {
    setState(() {
      _plotPoints = _evaluator.plotFunction(_controller.tex);
    });
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text('Function Plotter')),
      body: MathKeyboardViewInsets(
        child: Column(
          children: [
            Padding(
              padding: EdgeInsets.all(16),
              child: MathField(
                controller: _controller,
                decoration: InputDecoration(
                  labelText: 'f(x) =',
                  border: OutlineInputBorder(),
                  suffixIcon: IconButton(
                    icon: Icon(Icons.refresh),
                    onPressed: _updatePlot,
                  ),
                ),
              ),
            ),
            Expanded(
              child: CustomPaint(
                painter: FunctionPainter(_plotPoints),
                size: Size.infinite,
              ),
            ),
          ],
        ),
      ),
    );
  }
}
```

### 2. Custom Variables

Allow specific variables:

```dart
class CustomVariables extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text('Custom Variables')),
      body: MathKeyboardViewInsets(
        child: Padding(
          padding: EdgeInsets.all(16),
          child: MathField(
            keyboardType: MathKeyboardType.expression,
            variables: ['x', 'y', 'z', 'a', 'b'],  // Specify allowed variables
            decoration: InputDecoration(
              labelText: 'Enter expression',
              helperText: 'Available variables: x, y, z, a, b',
              border: OutlineInputBorder(),
            ),
          ),
        ),
      ),
    );
  }
}
```

### 3. Custom Focus Management

Control focus programmatically:

```dart
class FocusManagement extends StatefulWidget {
  @override
  _FocusManagementState createState() => _FocusManagementState();
}

class _FocusManagementState extends State<FocusManagement> {
  late final MathFieldEditingController _controller1;
  late final MathFieldEditingController _controller2;
  late final FocusNode _focusNode1;
  late final FocusNode _focusNode2;

  @override
  void initState() {
    super.initState();
    _controller1 = MathFieldEditingController();
    _controller2 = MathFieldEditingController();
    _focusNode1 = FocusNode();
    _focusNode2 = FocusNode();
  }

  @override
  void dispose() {
    _controller1.dispose();
    _controller2.dispose();
    _focusNode1.dispose();
    _focusNode2.dispose();
    super.dispose();
  }

  void _moveToNextField() {
    _focusNode1.unfocus();
    _focusNode2.requestFocus();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text('Focus Management')),
      body: MathKeyboardViewInsets(
        child: Padding(
          padding: EdgeInsets.all(16),
          child: Column(
            children: [
              MathField(
                controller: _controller1,
                focusNode: _focusNode1,
                decoration: InputDecoration(
                  labelText: 'Field 1',
                  border: OutlineInputBorder(),
                ),
                onSubmitted: (_) => _moveToNextField(),
              ),
              SizedBox(height: 16),
              MathField(
                controller: _controller2,
                focusNode: _focusNode2,
                decoration: InputDecoration(
                  labelText: 'Field 2',
                  border: OutlineInputBorder(),
                ),
              ),
            ],
          ),
        ),
      ),
    );
  }
}
```

### 4. Locale Support (Decimal Separator)

Handle different decimal separators:

```dart
class LocaleSupport extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Localizations.override(
      context: context,
      locale: Locale('de', 'DE'),  // German locale uses comma as decimal separator
      child: Scaffold(
        appBar: AppBar(title: Text('Locale Support')),
        body: MathKeyboardViewInsets(
          child: Padding(
            padding: EdgeInsets.all(16),
            child: MathField(
              keyboardType: MathKeyboardType.numberOnly,
              decoration: InputDecoration(
                labelText: 'Enter number',
                helperText: 'Decimal separator: , (comma)',
                border: OutlineInputBorder(),
              ),
            ),
          ),
        ),
      ),
    );
  }
}
```

---

## Common Patterns

### 1. Quiz Question

```dart
class QuizQuestion extends StatefulWidget {
  final String question;
  final String correctAnswer;

  const QuizQuestion({
    required this.question,
    required this.correctAnswer,
  });

  @override
  _QuizQuestionState createState() => _QuizQuestionState();
}

class _QuizQuestionState extends State<QuizQuestion> {
  late final MathFieldEditingController _controller;
  bool? _isCorrect;

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

  void _checkAnswer() {
    final userAnswer = _controller.tex;
    setState(() {
      _isCorrect = _compareExpressions(userAnswer, widget.correctAnswer);
    });
  }

  bool _compareExpressions(String expr1, String expr2) {
    // Normalize and compare
    final normalized1 = expr1.replaceAll(r'\s+', '');
    final normalized2 = expr2.replaceAll(r'\s+', '');
    return normalized1 == normalized2;
  }

  @override
  Widget build(BuildContext context) {
    return Card(
      margin: EdgeInsets.all(16),
      child: Padding(
        padding: EdgeInsets.all(16),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            Text(
              widget.question,
              style: Theme.of(context).textTheme.headline6,
            ),
            SizedBox(height: 16),
            MathField(
              controller: _controller,
              decoration: InputDecoration(
                labelText: 'Your answer',
                border: OutlineInputBorder(),
                suffixIcon: _isCorrect != null
                    ? Icon(
                        _isCorrect! ? Icons.check_circle : Icons.cancel,
                        color: _isCorrect! ? Colors.green : Colors.red,
                      )
                    : null,
              ),
              onSubmitted: (_) => _checkAnswer(),
            ),
            SizedBox(height: 16),
            ElevatedButton(
              onPressed: _checkAnswer,
              child: Text('Check Answer'),
            ),
            if (_isCorrect != null)
              Padding(
                padding: EdgeInsets.only(top: 8),
                child: Text(
                  _isCorrect! ? 'Correct! ✓' : 'Incorrect ✗',
                  style: TextStyle(
                    color: _isCorrect! ? Colors.green : Colors.red,
                    fontWeight: FontWeight.bold,
                  ),
                ),
              ),
          ],
        ),
      ),
    );
  }
}
```

### 2. Multi-Step Problem

```dart
class MultiStepProblem extends StatefulWidget {
  final List<String> steps;

  const MultiStepProblem({required this.steps});

  @override
  _MultiStepProblemState createState() => _MultiStepProblemState();
}

class _MultiStepProblemState extends State<MultiStepProblem> {
  int _currentStep = 0;
  final List<MathFieldEditingController> _controllers = [];

  @override
  void initState() {
    super.initState();
    for (var i = 0; i < widget.steps.length; i++) {
      _controllers.add(MathFieldEditingController());
    }
  }

  @override
  void dispose() {
    for (var controller in _controllers) {
      controller.dispose();
    }
    super.dispose();
  }

  void _nextStep() {
    if (_currentStep < widget.steps.length - 1) {
      setState(() {
        _currentStep++;
      });
    } else {
      _submitProblem();
    }
  }

  void _submitProblem() {
    final answers = _controllers.map((c) => c.tex).toList();
    print('Submitted answers: $answers');
    // Process answers
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text('Step ${_currentStep + 1} of ${widget.steps.length}'),
      ),
      body: MathKeyboardViewInsets(
        child: Padding(
          padding: EdgeInsets.all(16),
          child: Column(
            children: [
              LinearProgressIndicator(
                value: (_currentStep + 1) / widget.steps.length,
              ),
              SizedBox(height: 16),
              Expanded(
                child: SingleChildScrollView(
                  child: Column(
                    crossAxisAlignment: CrossAxisAlignment.start,
                    children: [
                      Text(
                        widget.steps[_currentStep],
                        style: Theme.of(context).textTheme.headline6,
                      ),
                      SizedBox(height: 16),
                      MathField(
                        controller: _controllers[_currentStep],
                        autofocus: true,
                        decoration: InputDecoration(
                          labelText: 'Answer',
                          border: OutlineInputBorder(),
                        ),
                        onSubmitted: (_) => _nextStep(),
                      ),
                    ],
                  ),
                ),
              ),
              SizedBox(height: 16),
              Row(
                children: [
                  if (_currentStep > 0)
                    Expanded(
                      child: OutlinedButton(
                        onPressed: () {
                          setState(() {
                            _currentStep--;
                          });
                        },
                        child: Text('Previous'),
                      ),
                    ),
                  if (_currentStep > 0) SizedBox(width: 16),
                  Expanded(
                    child: ElevatedButton(
                      onPressed: _nextStep,
                      child: Text(
                        _currentStep == widget.steps.length - 1
                            ? 'Submit'
                            : 'Next',
                      ),
                    ),
                  ),
                ],
              ),
            ],
          ),
        ),
      ),
    );
  }
}
```

### 3. Expression Builder

```dart
class ExpressionBuilder extends StatefulWidget {
  @override
  _ExpressionBuilderState createState() => _ExpressionBuilderState();
}

class _ExpressionBuilderState extends State<ExpressionBuilder> {
  late final MathFieldEditingController _controller;
  final List<String> _history = [];

  @override
  void initState() {
    super.initState();
    _controller = MathFieldEditingController();
    _controller.addListener(_saveToHistory);
  }

  @override
  void dispose() {
    _controller.removeListener(_saveToHistory);
    _controller.dispose();
    super.dispose();
  }

  void _saveToHistory() {
    if (_controller.tex.isNotEmpty) {
      setState(() {
        _history.insert(0, _controller.tex);
        if (_history.length > 10) {
          _history.removeLast();
        }
      });
    }
  }

  void _loadFromHistory(String latex) {
    _controller.updateExpression(latex);
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text('Expression Builder')),
      body: MathKeyboardViewInsets(
        child: Row(
          children: [
            // History sidebar
            Container(
              width: 200,
              color: Colors.grey[200],
              child: ListView.builder(
                itemCount: _history.length,
                itemBuilder: (context, index) {
                  return ListTile(
                    title: Text(
                      _history[index],
                      style: TextStyle(fontSize: 12),
                      maxLines: 1,
                      overflow: TextOverflow.ellipsis,
                    ),
                    onTap: () => _loadFromHistory(_history[index]),
                  );
                },
              ),
            ),
            // Main editor
            Expanded(
              child: Padding(
                padding: EdgeInsets.all(16),
                child: MathField(
                  controller: _controller,
                  decoration: InputDecoration(
                    labelText: 'Build your expression',
                    border: OutlineInputBorder(),
                  ),
                ),
              ),
            ),
          ],
        ),
      ),
    );
  }
}
```

---

## Troubleshooting

### Common Issues

#### 1. Keyboard Not Showing

**Problem:** Math keyboard doesn't appear when field is focused.

**Solution:**
```dart
// Ensure MathKeyboardViewInsets wraps your Scaffold
MathKeyboardViewInsets(
  child: Scaffold(
    // Your content
  ),
)
```

#### 2. Keyboard Overlapping Content

**Problem:** Content not pushed up by keyboard.

**Solution:**
```dart
// Use resizeToAvoidBottomInset in Scaffold
Scaffold(
  resizeToAvoidBottomInset: true,  // Default, but ensure it's not false
  body: SingleChildScrollView(  // Make content scrollable
    child: // Your content
  ),
)
```

#### 3. Invalid LaTeX Output

**Problem:** Generated LaTeX doesn't render correctly.

**Solution:**
```dart
// Debug the LaTeX string
print('LaTeX: ${controller.tex}');

// Check for common issues:
// - Unmatched braces
// - Invalid commands
// - Missing arguments
```

#### 4. Expression Evaluation Fails

**Problem:** TeXParser throws errors.

**Solution:**
```dart
try {
  final expr = TeXParser(latex).parse();
  // Use expression
} on FormatException catch (e) {
  print('Parse error: ${e.message}');
  // Show error to user
} catch (e) {
  print('Unknown error: $e');
}
```

#### 5. Physical Keyboard Conflicts

**Problem:** Physical keyboard shortcuts don't work or conflict with system shortcuts.

**Solution:**
```dart
// The package handles most physical keyboard input automatically
// For custom handling:
Focus(
  onKeyEvent: (node, event) {
    // Custom key handling
    return KeyEventResult.ignored;  // Let package handle it
  },
  child: MathField(/* ... */),
)
```

### Performance Issues

#### Slow Rendering

```dart
// Wrap in RepaintBoundary
RepaintBoundary(
  child: MathField(/* ... */),
)

// Reduce expression complexity
// Limit history size
controller.maxHistorySize = 50;  // If supported
```

#### Memory Leaks

```dart
// Always dispose controllers and focus nodes
@override
void dispose() {
  _controller.dispose();
  _focusNode.dispose();
  super.dispose();
}
```

---

## Best Practices

1. **Always wrap Scaffold with MathKeyboardViewInsets** for proper keyboard behavior
2. **Dispose controllers and focus nodes** to prevent memory leaks
3. **Use MathFormField** for forms with validation
4. **Handle errors gracefully** when parsing/evaluating expressions
5. **Test on multiple screen sizes** (phone, tablet, desktop)
6. **Provide clear feedback** for invalid input
7. **Use autofocus sparingly** to avoid unexpected keyboard pops
8. **Validate user input** before evaluation/submission
9. **Cache expensive operations** (rendering, evaluation)
10. **Provide undo/redo** for better UX (when supported)

---

## Resources

- [Package Documentation](https://pub.dev/packages/math_keyboard)
- [Example App](https://github.com/simpleclub/math_keyboard/tree/main/math_keyboard/example)
- [Demo App](https://simpleclub.github.io/math_keyboard)
- [Engineering Specification](../docs/ENGINEERING_SPECIFICATION.md)
- [Architecture Guide](../docs/ARCHITECTURE.md)

---

**Last Updated:** December 2025  
**Version:** 1.0
