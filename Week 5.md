### Week 5: Mastering Layout & Structure

**Overview:**  
This week, we'll dive into effective layout techniques, including how to structure your views and separate layout concerns. We'll also learn how to create custom view modifiers and gain confidence in reviewing PR's.

## 1. Structuring Your View Hierarchy

So far, we've worked with simple views that have a limited number of child views, which helps keep the `body` property clean and easy to manage.

For example:

```swift
var body: some View {
    VStack {
        Text("Welcome")
        Button("Click Me") {
            print("hello")
        }
    }
}
```

But what happens when we need to nest more views, or when the layout becomes more complex? Without careful planning, this can lead to a tangled, hard-to-read view hierarchy.

For example:

```swift
struct TestView: View {
    var body: some View {
        VStack {
            Text("Header")
            HStack {
                Image(systemName: "star.fill")
                Text("Featured")
            }
            Divider()
            ScrollView {
                VStack {
                    ForEach(0..<10) { index in
                        HStack {
                            Text("Item \(index)")
                            Text("Double Item \(index * 2)")
                        }
                    }
                }
            }
            Text("Footer")
        }
        HStack {
            Text("Testing")
            Spacer()
            Button("Click Me") {
                print("hello")
            }
        }
        LazyHStack {
            ForEach(0..<10) { index in
                HStack {
                    Text("Item \(index)")
                    Text("Double Item \(index * 2)")
                } 
            }
        }
    }
}
```

This can quickly become difficult to manage, much like the **Massive View Controller** issue.

---

### A Better Approach

Instead of placing every view in the main `body` which is difficult to maintain, we can use a few strategies to keep the hierarchy readable. One key technique is **separating concerns** by breaking down complex views into smaller, reusable components.

### Choosing How to Separate a View

There are two common ways to break a subview out of your main view, and the choice depends on the complexity and reuse potential of the subview:

1. **Private Computed Properties / Functions**  
   Use this when the subview is **simple** and only used **within the same view**. It keeps your file tidy without exposing internal layout details.
   - Use a **private computed property** if the subview doesn’t require parameters.
   - Use a **private function** returning `some View` if the subview needs input. <br/><br/>

   > ✅ Great for small, screen-specific UI elements that don’t warrant a separate struct.


2. **Extracting to a Separate View Struct**  
   
   Use this when the subview is **complex**, **reused in multiple places**, or deserves its own logic or styling. <br/><br/>

   > ✅ Great for reusable components, isolated logic, or anything you'd want to test or preview independently.


### Private Computed Properties / Functions

When a view is specific to a screen and doesn’t need to be reused elsewhere, it’s a good idea to keep it private and scoped within the parent view. Rather than placing all of the layout directly in the `body`, we can extract small subviews into private computed properties.

This keeps the main view clean and focused, while isolating logic that doesn’t need to be exposed.

#### Example

Let’s say we want to extract the header from a view:

```swift
struct TestView: View {
    var body: some View {
        VStack {
            header
            // Other content
        }
    }

    private var header: some View {
        VStack {
            Text("Header")
            HStack {
                Image(systemName: "star.fill")
                Text("Featured")
            }
        }
    }
}
```

By defining `header` as a private computed property, we make the layout more readable while keeping the implementation details hidden from the outside world.

---

### Private Subviews with Parameters

Sometimes, a subview needs input. In those cases, we can define a private function that returns a view instead of using a computed property.

#### Original

```swift
ForEach(0..<10) { index in
    HStack {
        Text("Item \(index)")
        Text("Double Item \(index * 2)")
    }
}
```

#### Refactored

```swift
ForEach(0..<10) { index in
    itemView(for: index)
}

private func itemView(for index: Int) -> some View {
    HStack {
        Text("Item \(index)")
        Text("Double Item \(index * 2)")
    }
}
```

This makes it easier to reason about the structure and reuse logic if needed—without promoting these views to standalone components unless necessary.

### Where to Define Subviews

In our project, we typically use one of two patterns to define subviews that are only used within a single screen:

1. **Private Variables or Functions on the Main View** 
   
   This is the simplest option. Define private computed properties or functions directly inside your `View` struct.

   ```swift
   struct ExampleView: View {
       var body: some View {
           VStack {
               header
               content(for: 5)
           }
       }

       private var header: some View {
           Text("Header")
       }

       private func content(for value: Int) -> some View {
           Text("Value: \(value)")
       }
   }
   ```

2. **Private View Extension** 
   
   When the subviews start cluttering the main view, or you want to keep the `body` focused, you can move the private views into a separate `private extension` on the same file. This keeps the logic clean and separated.

   ```swift
   struct ExampleView: View {
       var body: some View {
           VStack {
               header
               content(for: 5)
           }
       }
   }

   private extension ExampleView {
       var header: some View {
           Text("Header")
       }

       func content(for value: Int) -> some View {
           Text("Value: \(value)")
       }
   }
   ```
---

### Extracting to a Separate View Struct

When a subview becomes **too complex**, or if it’s **reused across multiple screens**, it’s a good idea to extract it into its own `View` struct. This keeps your main view concise, improves readability, and promotes reusability.

Let’s say we have this inline view:

```swift
HStack {
    Text("Testing")
    Spacer()
    Button("Click Me") {
        print("hello")
    }
}
```

Rather than leaving this inside the main view, we can pull it out into its own struct:

```swift
struct ReusableRowView: View {
    var body: some View {
        HStack {
            Text("Testing")
            Spacer()
            Button("Click Me") {
                print("hello")
            }
        }
    }
}
```

Then use it just like any other SwiftUI view:

```swift
struct TestView: View {
    var body: some View {
        VStack {
            header
            ReusableRowView()
            // Other content
        }
    }
}
```

---

### Passing in Data or Actions

If your subview needs to respond to user interaction or display dynamic data, we can make it configurable with parameters:

```swift
struct ReusableRowView: View {
    let title: String
    let onTap: () -> Void

    var body: some View {
        HStack {
            Text(title)
            Spacer()
            Button("Click Me", action: onTap)
        }
    }
}
```

Then in your main body:

```swift
ReusableRowView(title: "Testing") {
    print("Button tapped")
}
```

---

### Refactored view

Applying everything we've learnt, we can go from the first example to something like this

```swift
struct TestView: View {
    var body: some View {
        VStack {
            header
            Divider()
            scrollContent
            footer
        }
        ReusableRowView()
        horizontalList
    }

    private var header: some View {
        VStack {
            Text("Header")
            HStack {
                Image(systemName: "star.fill")
                Text("Featured")
            }
        }
    }

    private var scrollContent: some View {
        ScrollView {
            VStack {
                ForEach(0..<10) { index in
                    itemView(for: index)
                }
            }
        }
    }

    private var footer: some View {
        Text("Footer")
    }

    private var horizontalList: some View {
        LazyHStack {
            ForEach(0..<10) { index in
                itemView(for: index)
            }
        }
    }

    private func itemView(for index: Int) -> some View {
        HStack {
            Text("Item \(index)")
            Text("Double Item \(index * 2)")
        }
    }

    private func handleButtonTap() {
        print("hello")
    }
}
```

Much more readable than before, but could do with some further refinement!

---

### 2. Creating Custom `ViewModifier`s

By now, we’ve used several of SwiftUI’s built-in view modifiers to style and layout our views:

```swift
.padding()
.foregroundStyle(.primary)
.frame(width: 200)
.background(Color.gray)
```

These modifiers help us keep our code clean and declarative. But what if we find ourselves wanting to make our own? That’s where creating a custom `ViewModifier` comes in.

---

### Why Use a Custom Modifier?

Custom modifiers are great when:

- You're repeating the same styling or layout logic.
- You want to group multiple modifiers into a single, reusable modifier.
- You want to encapsulate styling logic and give it a meaningful name.

---

### Creating a ViewModifier

To create a custom modifier, create a new struct that conforms to the `ViewModifier` protocol. This has only one requirement, which is a method called `body` that accepts whatever content it’s being given to work with, and must return `some View`.

Let’s say we often want text that has a headline font, white text, padding, and a blue background. Instead of repeating that everywhere, we can extract it into a modifier.

In a `ViewModifier`, the `content` parameter represents the view the modifier is being applied to. You can think of it as the input view, and your job is to return a transformed version of it.

```swift
struct HighlightedTextModifier: ViewModifier {
    func body(content: Content) -> some View {
        content
            .font(.headline)
            .foregroundColor(.white)
            .padding()
            .background(Color.blue)
            .cornerRadius(8)
    }
}
```

Here, content could be any view, like a `Text`, `Image`, or even a custom component. The modifier applies styling or layout changes on top of it, returning a new view that wraps or builds on the original.

We can now use it with the `modifier()` modifier on our view.

```swift
Text("Important!")
    .modifier(HighlightedTextModifier())
```

However, This doesn't look like what we're used to...

---

### Making It Cleaner with Extensions

While using `.modifier(...)` works, it’s not as elegant as SwiftUI’s usual syntax. To make your custom modifier easier to apply, you can wrap it in a `View` extension:

```swift
extension View {
    func highlighted() -> some View {
        self.modifier(HighlightedTextModifier())
    }
}
```

Now you can apply it like this:

```swift
Text("Important!")
    .highlighted()
```

This makes your custom styling reusable, expressive, and easy to maintain, just like any other SwiftUI modifier.

---

## 3. Reviewing SwiftUI PRs

As our codebase grows and more views are built using SwiftUI, it's important to develop confidence in reviewing pull requests.

### What to look for when reviewing a SwiftUI PR

#### ✅ **Clear View Hierarchies**  
Is the view composed of logical subviews, or is the `body` growing into a wall of nested brackets?

A helpful rule of thumb: if you’re seeing more than 3 closing brackets in a row like this

```swift
            }
        }
    }
}
```

...it’s likely time to extract part of that view into a private computed property or a separate subview for better readability and structure.

---

#### 📐 **Logical View Placement**  
Are the views placed in an order that makes sense visually and structurally?

For example, don’t always default to stacking everything in a `VStack`. If something is meant to be visually anchored (like a floating button), consider using `.overlay(alignment:)`, `.background`, or even `ZStack` with alignment to get the desired effect more elegantly.

---

#### 🧹 **Avoid Redundant Containers**  
Check for unnecessary `VStack`, `ZStack`, or `HStack` wrappers that don’t actually contribute to layout or styling.

For example:

```swift
VStack {
    VStack {
        Text("Hello")
    }
}
```

The outer `VStack` may be completely unnecessary. Stripping out redundant wrappers keeps the layout lean and easier to understand.

---

#### ⚒️ **Consistent Use of Modifiers**  
Are the view modifiers applied in a consistent and logical order?

SwiftUI applies modifiers in a specific sequence, and this order can affect layout and appearance. Here are some things to watch out for:

The order in which modifiers are applied can change how they affect the view. For example, applying `.padding()` before `.background(Color.red)` gives a different result than the other way around.

 A good rule of thumb is to apply layout-related modifiers (like padding and frames) first, followed by styling (like background or corner radius), and then more specific adjustments (like shadow or z-index).

```swift
 // Correct order
Text("Hello")
    .padding()
    .background(Color.blue)
    .cornerRadius(10)
```

```swift
// Incorrect order
Text("Hello")
    .background(Color.blue)
    .padding()
    .cornerRadius(10)
```

#### 🦾 **Reuse and Duplication**  
Are repeated patterns or views extracted into private views or helper methods?

If you notice the same view or a recurring pattern being used multiple times, it’s a good opportunity to suggest extraction.

For smaller, reusable components, propose using private subviews. For more complex or larger components, consider creating a new struct to improve readability and maintainability. 

#### 🎩 **Correct Use of Moneybox Components and Modifiers**  

Are they using `Text` when they should be using `MBText`?

Look for instances where standard SwiftUI components are used instead of the Moneybox-specific equivalents. If Moneybox offers a custom component or modifier, it’s best to use those for consistency and to take advantage of the shared styles and behaviors.

For example, instead of manually applying `.background` and `.cornerRadius` to a view, using the existing `.roundedView` modifier in the MBComponents library that can achieve the same result more efficiently.

#### ❓ **Management of Conditionals**

As per our guidelines, we recommend using `.map` instead of traditional `if` statements. This approach leads to cleaner, more concise code.

If you see it, suggest to use `.map` instead, for example:

```swift
// Before
if let url = icon.url {
    MBLottieView(source: .remote(url))
}

// After
icon.url.map {
    MBLottieView(source: .remote($0))
}
```

## 4. Let's go through an example

In this section, we'll create a custom `ViewModifier` together.

We'll create a `ViewModifier` that allows you to pass in a background color, a rounded rectangle with corner radius, and a border color. Each of these properties will be customizable, so you can reuse the modifier with different configurations.

<details>
    
    import SwiftUI

    struct RoundedViewModifier: ViewModifier {
        var backgroundColor: Color
        var cornerRadius: CGFloat
        var borderColor: Color

        func body(content: Content) -> some View {
            content
                .padding()
                .background(backgroundColor)
                .cornerRadius(cornerRadius)
                .overlay(
                    RoundedRectangle(cornerRadius: cornerRadius)
                        .stroke(borderColor, lineWidth: 2)
                )
        }
    }

    extension View {
        func roundedView(backgroundColour: UIColor = .red,
                                borderColour: UIColor? = nil,
                                cornerRadius: CGFloat = 16) -> some View {
            modifier(RoundedViewModifier(backgroundColour: backgroundColour,
                                         borderColour: borderColour,
                                         cornerRadius: cornerRadius))
        }
    }  

</details>