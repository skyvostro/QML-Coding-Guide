# QML Coding Guide

## Item 1: Code Style

This section provides details about how to format the order of properties, signals, and functions to make things easy on the eyes and quickly switch to related code block.

[QML object attributes](https://doc.qt.io/qt-5/qtqml-syntax-objectattributes.html) are always structured in the following order:

- Property declarations
- Signal declarations
- id (Optionally `id` property can be put right after the object decleration to make it easier to locate an object.)
- Object properties
- States
- Transitions
- Signal handlers
- Child objects
  - Visual Items
  - Qt provided non-visual items
  - Custom provided non-visual items
- `QtObject` for encapsulating private members
- JavaScript functions

```qml
Rectangle {

    property bool thumbnail: false // Property declarations
    property alias image: photoImage.source

    signal clicked // Signal declarations

    id: photo
    x: 20
    y: 20
    height: 150
    color: "gray" // Object properties
    width: { // Large bindings
        if (photoImage.width > 200) {
            photoImage.width;
        }
        else {
            200;
        }
    }
    states: State { // States
        name: "selected"
        PropertyChanges { target: border; color: "red" }
    }
    transitions: Transition { // Transitions
        from: ""; to: "selected"
        ColorAnimation { target: border; duration: 200 }
    }
    onSomeEvent: {
        
    }

    Rectangle { // Child objects - Visual Items
        id: border
        anchors.centerIn: parent; color: "white"

        Image { id: photoImage; anchors.centerIn: parent }
    }
    
    Timer { } // Child objects - Qt provided non-visual items
    
    MyCppObject { } // Child object - Custom provided non-visual items

    QtObject {

        property var privateProperty: null

        id: privates
    }

    function doSomething(x) { // JavaScript functions
        return x + photoImage.width
    }
}
```

### Signal Handler Ordering

When handling the signals attached to an `Item`, make sure to always leave `Component.onCompleted` to the last line.

```qml
// Wrong
Item {
	Component.onCompleted: {
	}
    onSomeEvent: {
    }
}

// Correct
Item {
    onSomeEvent: {
    }
    Component.onCompleted: {
	}
}
```

This is because it mentally makes for a better picture because `Component.onCompleted` is expected to be fired when the components construction ends.

------

If there are multiple signal hanlers in an `Item`, then the ones with least amount of lines are palced at the top. As the implementation lines increases, the handler also moves down. The only exception to this is `Component.onCompleted` signal, it is always placed at the bottom.

```qml
// Wrong
Item {
	onOtherEvent: {
        // Line 1
        // Line 2
        // Line 3
        // Line 4
	}
    onSomeEvent: {
        // Line 1
        // Line 2
    }
}

// Correct
Item {
    onSomeEvent: {
        // Line 1
        // Line 2
    }
	onOtherEvent: {
        // Line 1
        // Line 2
        // Line 3
        // Line 4
	}
}
```

### Property Ordering

The first property assignment must always be the `id` of the component. If you want to declare custom properties for a component, the declerations are always above the first property assignment.

```qml
// Wrong
Item {
    someProperty: false
    property int otherProperty: -1
    id: myItem
}

// Correct
Item {

	property int otherProperty: -1
	
    id: myItem
    someProperty: false
}
```

There's also a bit of predefined order for property assignments. The order goes as follows:

- id
- x
- y
- width
- height
- anchors

The goal here is to put the most obvious and most defining properties at the top for easy access and visibility. For example, for an `Image` you may decide to also put `sourceSize` above `anchors`.

------

If there are also property assignments along with signal handlers, make sure to always put property assignments above the signal handlers.

```qml
// Wrong
Item {
	onOtherEvent: {
	}
	someProperty: true
    onSomeEvent: {
    }
    x: 23
    y: 32
}

// Correct
Item {
    x: 23
    y: 32
    someProperty: true
    onOtherEvent: {
	}
    onSomeEvent: {
    }
}
```

It is usually harder to see the property assignments If they are mixed with signal handlers. That's why we are putting the assignments above the signal handlers.

### Function Ordering

Although there are no private and public functions in QML, you can provide a similar mechanism by wrapping the properties and and functions that are only supposed to be used internally in `QtObject `.

Public function implementations are always put at the very buttom of the file.

```qml
// Wrong
Item {
	
	function someFunction() {
	}

	someProperty: true
}

// Correct
Item {
    someProperty: true
    onOtherEvent: {
	}
    onSomeEvent: {
    }
    
    function someFunction() {   
	}
}
```

### Animations

When using any subclass of `Animation`, especially nested ones like `SequentialAnimation`, try to keep the `Animation` objects to one line for readibilty. You will benefit from keeping the animations as simple as possible since they are executed every frame, and also find it easier to see the execution of the animation in your head.

```qml
// Bad
SequentialAnimation {
    PropertyAction {
    	target: root
    	property: "visible"
    	value: true
    }
    
    NumberAnimation {
    	target: root
        property: "opacity"
        duration: root.animationDuration
        from: 0
        to: 1
    }
}

// Good. This is easier to read as your eyes find it easy to detect things horizontally rather than vertically.
SequentialAnimation {
    PropertyAction { target: root; property: "visible"; value: true }
    NumberAnimation { target: root; property: "opacity"; duration: root.animationDuration; from: 0; to: 1 }
}
```

### Giving Components `id`s

If a component does not need to be accessed for a functionality, avoid setting the `id` property. This way you don't clutter the namespace with unused `id`s and you'll be less likely to run into duplicate `id` problem.

It is a good idea to use max 3-4 character abbreviation for the `id`s so that whe you are looking for a certain component, say a `TextBox`, it will be easier to list the IDs of all the text boxes by just typing `tb`.

The schema would be `[COMPONENT_NAME][COMPONENT_DESCRIPTION]`, e.g `tbEmail`, `btnLogIn`

```qml
TextBox {
    id: tbEmail
}

Button {
    id: btnSubmit
}

CheckBox {
    id: cbAgreement
}
```

Make sure that the top most component in the file always has `root` as its `id`.

### Property Assignments

When assigning grouped properties, always prefer the dot notation If you are only altering just one property. Otherwise, always use the group notation.

```qml
Image {
    anchors.left: parent.left // Dot notation
    sourceSize { // Group notation
        width: 32
        height: 32
    }
}
```

When you are assigning the component to a `Loader`'s `sourceComponent` in different places in the same file, consider using the same implementation. For example, in the following example there are two instances of the same component. If both of those `SomeSpecialComponent` are meant to be identical it is a better idea to wrap `SomeSpecialComponent` in a `Component`.

```qml
// BEGIN bad.
Loader {
    id: loaderOne
    sourceComponent: SomeSpecialComponent {
        text: "Some Component"
    }
}

Loader {
    id: loaderTwo
    sourceComponent: SomeSpecialComponent {
        text: "Some Component"
    }
}
// END bad.

// BEGIN good.
Loader {
    id: loaderOne
    sourceComponent: specialComponent
}

Loader {
    id: loaderTwo
    sourceComponent: specialComponent
}

Component {
    id: specialComponent
    
    SomeSpecialComponent {
        text: "Some Component"
    }
}
// END good.
```

This ensures that whenever you make a change to `specialComponent` it will take effect in all of the `Loader`s. In the bad example, you would have to duplicate the same change.

## Import Statements

Imports are take time in QML. And If you are developing for a device with low system specifications, then you will want to optimize as much as possible. In that case, try to minimize the number of imports you use in your QML file.

If you are also importing a JavaScript file, make sure to not include the same module in both the QML file and the JavaScript file. JavaScript files share the imports from the QML file so you can take advantage of that. Note that Qt Create does not provide code completion for the modules that you import in the QML file.

If you are not making use of the imported module in the QML file, consider moving the import statement to the JavaScript file. But note that once you import something in the JavaScript file, the imports will no longer be shared. For the complete rules see [here](https://doc.qt.io/qt-5/qtqml-javascript-imports.html#imports-within-javascript-resources).

Alternatively, you can use `Qt.include()` which copies the contens of the included file and you will not have to worry about the import sharing rules.

#### Import Order

When importing other modules, use the following order;

- Qt modules
- Third party modules
- Local C++ module imports
- QML folder imports



## Item 2: Bindings

Bindings are a powerful tool when used responsibly. Bindings are evaluated whenever a property it depends on changes and this may result in poor performance or unexpected behaviours. Even when the binding is simple, its consequence can be expensive. For instance, a binding can cause the position of an item to change and every other item that depends on the position of that item or is anchored to it will also update its position.

So consider the following rules when you are using bindings.

### Reduce the Number of Bindings

When using bindings, there are bound to be cases where a single changed signal can be used to update multiple values. Consider the following example:

```qml
Rectangle {
    id: root
    color: mouseArea.pressed ? "red" : "yellow"
    
    Text {
        anchors.centerIn: parent
        text: mouseArea.pressed ? "Red Color" : "Yellow Color"
    }
    
    MouseArea {
	    id: mouseArea
    	anchors.fill: parent
    }
}
```

Now, the bindings are simple in this case so just imagine that they were not that simple and we had a bigger app. As you can see there are two bindings and each is executed when the user presses the mouse button.

We can rewrite that as follows to reduce the number of bindings to only one.

```qml
Rectangle {
    id: root
    
    Text {
	    id: label
        anchors.centerIn: parent
    }
    
    MouseArea {
	    id: mouseArea
    	anchors.fill: parent
    	onPressedChanged: {
            if (pressed) {
                root.color = "red";
                label.text = "Red Color";
            }
            else {
            	root.color = "yellow";
                label.text = "Yellow Color";
            }
    	}
    }
}
```

Now whenever the user presses the mouse button, only one block will be excuted for two of those expected outcomes.

Alternatively, you can use `Connections` to connect to a particular signal for an object and update the proeprties in the signal handler. This canse is particularly useful when you are using `Loader`s.

### Making `Connections`

`Connections` object is used to handle signals from artbitrary `QObject` derived classes in QML. One thing to keep in mind when using connections is the default value of `target` property of the `Connections` is its parent if not explicitly set to something else. If you are setting the target after dynamically creating a QML object, you might want to set the `target` to `null` otherwise you might get signals that are not meant to be handled.

```qml
// Bad
Item {
    id: root
    onSomeEvent: {
        // Set the target of the Connections.
    }
    
    Connections {
	    // Notice that target is not set so it's implicitly set to root.
        onWidthChanged: {
            // Do something. But since Item also has a width property we may handle the change for root untill the target is set explicitly.
        }
    }
}

// Good
Item {
    id: root
    onSomeEvent: {
        // Set the target of the Connections.
    }
    
    Connections {
    	target: null // Good. Now we won't have the same problem.
        onWidthChanged: {
            // Do something. Only handles the changes for the intended target.
        }
    }
}
```

### Use `Binding` Object 

`Binding`'s `when` property can be used to enable or disable a binding expression bepending on a condition. If the binding that you are using is complex and does not need to be executed everytime a property changes, this is a good idea to reduce the binding execution count.

Using the same example above, we can rewrite it as follows using a `Binding` object.

```qml
Rectangle {
    id: root
    
    Binding on color {
        when: mouseArea.pressed
        value: mouseArea.pressed ? "red" : "yellow"
    }
    
    MouseArea {
	    id: mouseArea
    	anchors.fill: parent
    }
}
```

Again, this is a really simple example to get the point out. In a real life situation, you would not get more benefit from using `Binding` object in this case unless the binding expression is expensive (e.g It changes the an item's `anchor` which causes a whole chain reaction and causes other items to be repositioned.).

### KISS It

You are problably already familiar with the [KISS principle](https://en.wikipedia.org/wiki/KISS_principle). QML supports optimization of binding expressions. Optimized bindings do not require a JavaScript environment hence it runs faster. The basic requirement for optimization of bindings is that the type  information of every symbol accessed must be known at compile time.

So, avoid accessing `var` properties. You can see the full list of prerequisites of optimized bindings [here](https://doc.qt.io/qt-5/qtquick-performance.html#bindings).

### Be Lazy

There may be cases where you don't need the binding immediately but when a certain condition is met. By lazily creating a binding, you can avoid unnecessary executions. To create a binding during runtime, you can use `Qt.binding()`.

```qml
Item {
    property int edgePosition: 0

    Component.onCompleted: {
        if (checkForSomeCondition() == true) {
            // bind to the result of the binding expression passed to Qt.binding()
            edgePosition = Qt.binding(function() { return x + width })
        }
    }
}
```

You can also use `Qt.callLater` to reduce the redundant calls to a function.

### Avoid Unnecessary Re-Evaluations

If you have a loop or process where you update the value of the property, you may want to use a temporary local variable where you accumulate those changes and only report the last value to the property. This way you can avoid triggering re-evaluation of binding expressions during the intermediate stages of accumulation.

Here's a bad example straigth from Qt documentation:

```qml
import QtQuick 2.3

Item {

    property int accumulatedValue: 0
    
    id: root
    width: 200
    height: 200
    Component.onCompleted: {
        var someData = [ 1, 2, 3, 4, 5, 20 ];
        for (var i = 0; i < someData.length; ++i) {
            accumulatedValue = accumulatedValue + someData[i];
        }
    }

    Text {
        anchors.fill: parent
        text: root.accumulatedValue.toString()
        onTextChanged: console.log("text binding re-evaluated")
    }
}
```

And here is the proper way of doing it:

```qml
import QtQuick 2.3

Item {
	
	property int accumulatedValue: 0

    id: root
    width: 200
    height: 200
    Component.onCompleted: {
        var someData = [ 1, 2, 3, 4, 5, 20 ];
        var temp = accumulatedValue;
        for (var i = 0; i < someData.length; ++i) {
            temp = temp + someData[i];
        }
        
        accumulatedValue = temp;
    }

    Text {
        anchors.fill: parent
        text: root.accumulatedValue.toString()
        onTextChanged: console.log("text binding re-evaluated")
    }
}
```
