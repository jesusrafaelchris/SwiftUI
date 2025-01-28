# Week 2: Managing State in SwiftUI

## Overview
This week, we’ll explore how state is managed and updated within SwiftUI views. We'll also look at how state is passed between views and how SwiftUI components, like Button, interact with state to create dynamic user experiences.

SwiftUI provides powerful property wrappers for managing state, allowing us to seamlessly observe, update, and render data. These tools make state management intuitive and easy.

## 1. How do we change state?

SwiftUI views are a function of their state, which means that the way the our UI looks and behaviour are determined by the state in our view.

Let's start with a simple example:

```swift
struct CounterView: View {
    var count: Int = 0
    
    var body: some View {
        VStack {
            Text("Count: \(count)")
                .font(.largeTitle)
            
            Button("Increment") {
                count += 1
            }
            .buttonStyle(.borderedProminent)
        }
    }
}

```

At first glance, this code seems reasonable. When we tap the button, we increment the count, and the updated value should be displayed in the `Text` component. However, this isn't valid SwiftUI code.

Since `CounterView` is a struct, it's immutable by default, and we can't change its properties directly. To modify a property within a struct, we would typically need to use the mutating keyword. But in SwiftUI, this approach isn't allowed because the view's body must be declarative and reactive, not imperative.

So, how can we manage this state change?


## 2. State Properties: @State

SwiftUI introduces the @State property wrapper to handle a view’s local state. It enables us to manage small, transient pieces of data specific to a single view.

### Why @State?

@State allows us to work around the limitation of structs: we know we can’t change their properties because structs are immutable, but @State allows that value to be stored separately by SwiftUI in a place where it can be modified.

This means that when the state changes, SwiftUI automatically re-renders the view to reflect those changes.

Here's a simple example, where we declare a state property of count.

### Example:

```swift
    @State private var count: Int = 0
}
```
We can use the `@State` property wrapper to designate a property as a state property. 

 **Note:** @State is specifically designed for simple properties that are stored internally in one view. It’s therefore most often a good idea to keep State-wrapped properties private, which ensures that they’ll only be mutated within that view’s body (attempting to mutate them elsewhere will actually cause a runtime crash)

In SwiftUI, state can be modified in response to user actions, such as tapping a button. 

### Further Example:

```swift
struct CounterView: View {
    @State private var count: Int = 0
    
    var body: some View {
        VStack {
            Text("Count: \(count)")
                .font(.largeTitle)
            
            Button("Increment") {
                count += 1
            }
            .buttonStyle(.borderedProminent)
        }
    }
}
```

### What’s Happening Here?
- The `@State` wrapper tracks the `count` variable.
- Each time the button is tapped, the `count` value is updated, and SwiftUI automatically re-renders the view with the new value.


### Passing State to Children

When we pass state to child views, the parent typically controls the state, while the child can only read it. This is known as parent-controlled state—the child view can access the data but cannot modify it.

### Example:

```swift
struct ParentView: View {
  @State private var text = "hello"
  
  var body: some View {
    ChildView(text: text)
  }
}

struct ChildView: View {
  let text: String
  
    var body: some View {
    Text(text)
  }
}
```

In this example, the ParentView owns the text state and passes it to the ChildView. The ChildView receives the value of text but cannot modify it.

## 3. State Properties: @Binding

Sometimes, we want a child view to modify state that is managed by its parent. In such cases, SwiftUI provides the @Binding property wrapper.

@Binding creates a two-way connection between a parent’s state and the child view. This allows the child to read the state and also modify it, while keeping the source of truth in the parent.

## How @Binding Works

With @Binding, we declare that a property in the child view is not owned by the child but instead references a value managed elsewhere — typically the parent. This ensures that updates in the child are reflected in the parent, and vice versa.


### Example: Using @Binding for Two-Way Communication
Here’s how to use @Binding to enable a child view to update a parent’s state:

 ### Example:

```swift
struct ParentView: View {
    @State private var isOn: Bool = false
    
    var body: some View {
        VStack {
            ToggleView(isOn: $isOn) // Passes the binding to the child
            Text(isOn ? "Switch is ON" : "Switch is OFF")
                .font(.headline)
        }
        .padding()
    }
}

struct ToggleView: View {
    @Binding var isOn: Bool // Declares a binding property
    
    var body: some View {
        Toggle("Enable Option", isOn: $isOn)
            .padding()
    }
}

```

###  What’s Happening Here:
- The `ParentView` manages the `isOn` state with `@State`.
- The `ParentView` passes a binding (`$isOn`) to the `ToggleView` child. The `$` symbol creates a binding from the `@State` property.
- The `ToggleView` declares a `@Binding` property, isOn, and uses it to control the Toggle component.
- When the toggle is switched, the `isOn` value is updated in the parent view, and the UI updates to reflect the new state.

**Key learning**
- @State in Parent, @Binding in Child:
Use @State in the parent view to manage the source of truth for the state. Use @Binding in the child view to allow it to read and modify that state. This ensures a clear ownership hierarchy while enabling two-way communication.

- The $ Symbol for Binding:
The $ symbol is used to bind the @State property to the @Binding property, creating a shared link between the parent and child. Any updates in one are automatically reflected in the other.

 ### Example recap:

```swift
struct ParentView: View {
    @State private var isOn: Bool = false // Source of truth in parent

    var body: some View {
        ChildView(isOn: $isOn) // Pass binding to the child
    }
}

struct ChildView: View {
    @Binding var isOn: Bool // Binding property in child

    var body: some View {
        Toggle("Switch", isOn: $isOn) // Binding allows updates
    }
}
```

## 4. Components

### Button

In SwiftUI, Button is one of the most fundamental components for handling user interactions. It is used to trigger actions or events when tapped. SwiftUI provides multiple ways to create and style buttons using different initializers, allowing flexibility in how you define their appearance and behavior.


A Button is initialized with:

- An action: A closure defining what should happen when the button is tapped.
- A label: The content (text, image, or custom view) displayed on the button.


### Button Overview

In SwiftUI, `Button` is one of the most fundamental components for handling user interactions. It is used to trigger actions or events when tapped. SwiftUI provides multiple ways to create and style buttons using different initializers, allowing flexibility in how you define their appearance and behavior.


#### **Basic Button with Text Label**
This is the simplest form of a button, displaying a text label and an action.

```swift
Button("Tap Me") {
    print("Button tapped!")
}
```

####  **Basic Button with Custom View**
When designing a button, you can use a custom view as the label to allow for more flexibility in the button's appearance. This is especially useful when you want to wrap a group of views (like text, images, or shapes) into a tappable area that performs an action.

```swift
Button(action: {
    print("SwiftUI is so amazing!")
}) {
    VStack {
        Image(systemName: "heart.fill")
            .font(.largeTitle)
            .foregroundColor(.red)
        Text("Like")
            .font(.headline)
    }
}
```

### Why Use `Button` Over `onTapGesture`?

While you can use `onTapGesture` to add actions to any view, **`Button`** is the preferred approach in most cases, particularly for accessibility reasons:

- **Accessibility Benefits**:  
  SwiftUI automatically makes `Button` accessible by:
  - Providing tap actions for screen readers.
  - Announcing the button's label via VoiceOver.
  - Adjusting its size and tap area for better usability.

- **Styling and Interaction**:  
  `Button` provides built-in support for visual and tactile feedback (like highlighting the button when pressed), which enhances the user experience.

#### Example of `onTapGesture`:

```swift
VStack {
    Image(systemName: "heart.fill")
        .font(.largeTitle)
        .foregroundColor(.red)
    Text("Like")
        .font(.headline)
}
.onTapGesture {
    print("SwiftUI is so amazing!")
}
```

While `onTapGesture` works for simple interactions, it lacks the accessibility features and system-level enhancements of `Button`. **Use `Button` whenever possible for better usability and accessibility.**

### Toggling between state

SwiftUI provides a simple and elegant way to toggle between two states using the `.toggle()` method. This is particularly useful for managing boolean state variables, such as showing or hiding content based on user interaction.

```swift
struct ContentView: View {
    @State private var showDetails = false

    var body: some View {
        VStack(alignment: .leading) {
            Button("Show details") {
                showDetails.toggle()
            }

            if showDetails {
                Text("This is crazy talk!")
                    .font(.largeTitle)
            }
        }
    }
}
```

#### When to Use `.toggle()`?

The `.toggle()` method is ideal for simple boolean state transitions, such as:

- Expanding or collapsing sections.
- Switching between "on" and "off" states.
- Showing or hiding elements dynamically.

### Views That Take in `@Binding`

SwiftUI provides several views that accept a `@Binding` property to share and modify state between views. These views are designed to seamlessly update the state when user interactions occur, making it easier to create dynamic and responsive UIs.

#### 1. **TextField**
The `TextField` view binds to a string property, allowing it to dynamically update as the user types.

##### Example:
```swift
struct TextFieldExample: View {
    @State private var name = ""

    var body: some View {
        VStack {
            TextField("Enter your name", text: $name)
                .textFieldStyle(.roundedBorder)
                .padding()

            Text("Hello, \(name)!")
        }
    }
}
```

- **Binding Property**: `text`  
  The `TextField` updates the `name` property in real-time as the user types.

#### 2. **Toggle**
The `Toggle` view binds to a boolean property, enabling it to update the state when toggled on or off.

##### Example:
```swift
struct ToggleExample: View {
    @State private var isOn = false

    var body: some View {
        VStack {
            Toggle("Enable Notifications", isOn: $isOn)
                .padding()

            Text(isOn ? "Notifications Enabled" : "Notifications Disabled")
        }
    }
}
```

- **Binding Property**: `isOn`  
  The `Toggle` view updates the `isOn` property when the user interacts with it.


#### 3. **Slider**
The `Slider` view binds to a numeric property (`Double`), allowing it to update the value as the user slides.

##### Example:
```swift
struct SliderExample: View {
    @State private var value: Double = 0.5

    var body: some View {
        VStack {
            Slider(value: $value, in: 0...1)
                .padding()

            Text("Value: \(value, specifier: "%.2f")")
        }
    }
}
```

- **Binding Property**: `value`  

#### 4. **Stepper**
The `Stepper` view binds to a numeric property, incrementing or decrementing its value when tapped.

##### Example:
```swift
struct StepperExample: View {
    @State private var count = 0

    var body: some View {
        VStack {
            Stepper("Count: \(count)", value: $count)
                .padding()
        }
    }
}
```

- **Binding Property**: `value`  

#### 5. **Picker**
The `Picker` view binds to a property that stores the selected value.

##### Example:
```swift
struct PickerExample: View {
    @State private var selectedColor = "Red"
    let colors = ["Red", "Green", "Blue"]

    var body: some View {
        Picker("Select a color", selection: $selectedColor) {
            ForEach(colors, id: \.self) {
                Text($0)
            }
        }
        .pickerStyle(.segmented)

        Text("Selected Color: \(selectedColor)")
    }
}
```

- **Binding Property**: `selection`  

In each of these built-in SwiftUI views—such as `TextField`, `Toggle`, `Slider`, `Stepper`, and `Picker`—the **child view** (e.g., the `TextField` or `Toggle`) directly updates the **state property** managed by the **parent view**.  

This two-way communication is possible because of the `@Binding` property wrapper. The `$` prefix is used to pass the **binding reference** of the state property to the child view, rather than its value. This ensures that any changes made by the child are reflected in the parent, and the view hierarchy stays in sync.


## 5. Let's try an example

Now that we’ve explored @State, @Binding, and some key SwiftUI components, let’s try a fun example to solidify these concepts.

<img src="https://github.com/user-attachments/assets/40c61111-75a2-4503-a0dd-a32024db6f0c" width=250/>

<details>
  <summary>Click to reveal the answer</summary>
  import SwiftUI

struct ParentView: View {
    @State private var age = 0
    @State private var email: String = ""
    @State private var password: String = ""
    @State private var isOver18: Bool = false
    
    var body: some View {
        VStack {
            Text("Enter your information")
                .font(.title)
                .frame(maxWidth: .infinity, alignment: .leading)
                .padding(.bottom, 32)
            VStack(spacing: 20) {
                TextField("Email", text: $email)
                    .textFieldStyle(.roundedBorder)
                SecureField("Password", text: $password)
                    .textFieldStyle(.roundedBorder)
                Toggle("Do you love SwiftUI?", isOn: $isOver18)
                ChildView(age: $age)
            }
        }
        .frame(maxHeight: .infinity, alignment: .center)
        .padding()
        .overlay(alignment: .bottom) {
            Button {
                print(email, password, isOver18, age)
            } label: {
                HStack {
                    Text("Confirm")
                    Spacer()
                    Image(systemName: "chevron.right")
                }
                .foregroundStyle(.white)
                .padding()
                .background {
                    RoundedRectangle(cornerRadius: 12)
                        .foregroundStyle(.red)
                }
            }
            .padding()
        }
    }
}

struct ChildView: View {
    @Binding var age: Int
    
    var body: some View {
        HStack(spacing: 12) {
            Text("Age in years: \(age)")
            Spacer()
            Button {
                age += 1
            } label: {
                Text("+")
                    .foregroundStyle(.white)
                    .padding()
                    .background {
                        Circle()
                    }
            }
            Button {
                age -= 1
            } label: {
                Text("-")
                    .padding()
                    .foregroundStyle(.white)
                    .background {
                        Circle()
                    }
            }
        }
    }
}

#Preview {
    ParentView()
}

</details> 
