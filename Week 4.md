# Week 4: Advanced Layout Techniques 

## Overview  
This week, we’ll dive into advanced layout techniques, including using ForEach to display dynamic content, working with List, LazyVStack + LazyHStack, ScrollView, and taking a first look at view lifecycle events.

## 1. Creating Views in a Loop with ForEach

In SwiftUI, you often need to loop over a sequence to create views. This is done using `ForEach`.

`ForEach` in SwiftUI is a view struct, meaning you can return it directly from your view body. It’s different from the `forEach()` method on Swift’s sequences.

### How does it work?

You provide `ForEach` with an array of items and an identifier to help SwiftUI track each item uniquely, ensuring it updates correctly when values change. You also provide a closure to create a view for each item.

For example, here’s how you can loop over the range `1...10` and display each number in a `Text` view:

```swift
VStack(alignment: .leading) {
    ForEach(1...10, id: \.self) { number in
        Text("\(number)")
    }
}
```

### What is `id: \.self`?

In SwiftUI, `id: \.self` uniquely identifies each item in a collection. When you use `ForEach`, SwiftUI needs a way to track which view corresponds to which item in the array. By providing `id: \.self`, you're telling SwiftUI to use the element itself as its unique identifier.

This works similarly to how `cellForRow(at:)` or `cellForItem(at:)` operates in UIKit, where the index path is used to identify and retrieve specific cells in a `UITableView` or `UICollectionView`. With `id: \.self`, SwiftUI automatically handles the mapping between data and views, without requiring explicit index management.

But where does this `\.self` identifier come from?

### Hashable & Identifiable

In SwiftUI, `Hashable` and `Identifiable` are used to track and uniquely identify items in a collection.

`ForEach` needs to know how to uniquely identify each item, otherwise, you'll encounter the following error:

```swift
Generic struct 'ForEach' requires that 'Item' conform to 'Hashable'
```

### What do `Hashable` and `Identifiable` mean?

- **`Hashable`**: Allows items to be compared for uniqueness, enabling efficient updates. It's especially useful when the item itself is the identifier, like with `id: \.self` in `ForEach`. Simple types like `Int` and `String` automatically conform to `Hashable`.

- **`Identifiable`**: Provides a stable, unique identifier for each item via the `id` property. SwiftUI uses this to track items and ensure they are correctly identified, even if the collection changes.

When using simple types like `String` or `Int` in `ForEach`, they already conform to `Hashable`, so SwiftUI can automatically use `id: \.self`. However, for more complex types, this automatic synthesis isn’t available. In these cases, you need to explicitly conform to `Identifiable` and implement an `id` property.

### Example 1: Conforming to `Identifiable`

Here’s an example where we conform a custom struct to `Identifiable` and use `id: \.id`:

```swift
struct Item: Identifiable {
    var id: Int
    var name: String
}

struct ContentView: View {
    let items = [
        Item(id: 1, name: "Apple"),
        Item(id: 2, name: "Banana"),
        Item(id: 3, name: "Cherry")
    ]

    var body: some View {
        VStack {
            ForEach(items, id: \.id) { item in
                Text(item.name)
                    .padding()
            }
        }
    }
}
```

### Example 2: Conforming to `Hashable`

If your data is made up of `Hashable` items, you can make the entire struct conform to `Hashable`, and let SwiftUI synthesize the `id`:

```swift
struct Item: Hashable {
    var id: Int
    var name: String
}

struct ContentView: View {
    let items = [
        Item(id: 1, name: "Apple"),
        Item(id: 2, name: "Banana"),
        Item(id: 3, name: "Cherry")
    ]

    var body: some View {
        VStack {
            ForEach(items, id: \.self) { item in
                Text(item.name)
                    .padding()
            }
        }
    }
}
```

- **`id: \.self`** works with simple types that conform to `Hashable`, allowing SwiftUI to automatically track and update items.
- For custom structs, you need to conform to **`Identifiable`** by providing a unique `id` property.
- **`Hashable`** allows efficient updates by enabling comparisons of items, while **`Identifiable`** provides a stable unique identifier.


## 2. LazyVStack and LazyHStack

In previous examples, we've seen how to use `VStack` and `HStack` to stack views. However, these stacks load all their contents upfront, which can slow down performance and waste memory when used inside a `ScrollView`, especially with large datasets.

`LazyVStack` and `LazyHStack` are alternatives that solve this problem. They create vertical and horizontal stacks of views, respectively, but with one key difference: they only load the views that are currently visible on the screen. This lazy loading mechanism helps improve performance and memory usage, making them ideal for large or dynamic content.

In UIKit terms, think of `LazyVStack` and `LazyHStack` as similar to dequeuing reusable cells in a `UITableView` or `UICollectionView`. Just as these views only create and reuse cells that are visible on the screen, `LazyVStack` and `LazyHStack` only create the views that are currently in view, efficiently managing memory and performance as you scroll. 

### LazyVStack

`LazyVStack` arranges views vertically and only creates the views that are visible in the scrollable area. As you scroll, new views are created dynamically, and off-screen views are discarded.

```swift
ScrollView {
    LazyVStack {
        ForEach(1...1000, id: \.self) { number in
            Text("Item \(number)")
                .padding()
        }
    }
}
```

Here, `LazyHStack` loads only the visible views as the user scrolls `Vertically`.

### LazyHStack

`LazyHStack` works similarly to `LazyVStack`, but it arranges views horizontally instead.

```swift
ScrollView(.horizontal) {
    LazyHStack {
        ForEach(1...1000, id: \.self) { number in
            Text("Item \(number)")
                .padding()
        }
    }
}
```

Here, `LazyHStack` loads only the visible views as the user scrolls `Horizontally`.

### When to Use LazyVStack and LazyHStack

- Use `LazyVStack` or `LazyHStack` when you have large datasets or dynamic content that doesn’t need to be fully loaded into memory at once.
- These stacks are most effective in `ScrollView`, where they allow efficient scrolling by only loading the views that are currently visible on the screen.

### Summary

- **`LazyVStack`**: A vertically scrollable stack that loads views as they come into view.
- **`LazyHStack`**: A horizontally scrollable stack that loads views lazily.
- Both improve performance and memory management when dealing with large datasets in scrollable views.

## 3. Using ScrollViews

In SwiftUI, `ScrollView` is a container view that allows content to be scrollable when it exceeds the available space on the screen. It’s commonly used for building screens where you expect content to be larger than the screen, such as lists of items, images, or other complex layouts.

### Basic ScrollView

A simple `ScrollView` lets you scroll through content vertically or horizontally. When the content inside the `ScrollView` exceeds the bounds of the parent view, the scrollability comes into play.

#### Vertical ScrollView

A vertical `ScrollView` enables scrolling along the vertical axis:

```swift
ScrollView {
    VStack {
        ForEach(1...100, id: \.self) { number in
            Text("Item \(number)")
                .padding()
        }
    }
}
```

In this example, `VStack` holds the list of `Text` views, and because we’re inside a `ScrollView`, the user can scroll vertically through the items.

Here `ScrollView {` defaults to the `.vertical` direction.

#### Horizontal ScrollView

For horizontal scrolling, simply specify the `.horizontal` direction:

```swift
ScrollView(.horizontal) {
    HStack {
        ForEach(1...100, id: \.self) { number in
            Text("Item \(number)")
                .padding()
        }
    }
}
```

In this case, the items are placed in a `HStack`, and the scroll view enables horizontal scrolling. This is useful when you have large sets of horizontally arranged content, like images or cards.

### Combining ScrollView with Lazy Stacks

When working with large data sets inside a `ScrollView`, combining it with `LazyVStack` or `LazyHStack` ensures that only the views currently visible are rendered, improving performance. For example, using `LazyVStack` inside a `ScrollView`:

```swift
ScrollView {
    LazyVStack {
        ForEach(1...1000, id: \.self) { number in
            Text("Item \(number)")
                .padding()
        }
    }
}
```

Here, as the user scrolls through the list, new views are lazily loaded, improving both performance and memory management.

### UIKit vs SwiftUI

Now, at last, we can create advanced views, similar to what we're used to with `UITableView` and `UICollectionView` in UIKit. However, with SwiftUI, it's much easier and faster to implement these types of views.

In UIKit, you need to set up a `UICollectionView`, define a layout, and implement the data source and delegate methods manually.

```swift
import UIKit

class ViewController: UIViewController, UICollectionViewDataSource {
    var collectionView: UICollectionView!
    let data = ["Item 1", "Item 2", "Item 3", "Item 4"]
    
    override func viewDidLoad() {
        super.viewDidLoad()
        let layout = UICollectionViewFlowLayout()
        layout.itemSize = CGSize(width: 100, height: 100)
        
        collectionView = UICollectionView(frame: self.view.bounds, collectionViewLayout: layout)
        collectionView.dataSource = self
        collectionView.register(UICollectionViewCell.self, forCellWithReuseIdentifier: "cell")
        self.view.addSubview(collectionView)
    }

    func collectionView(_ collectionView: UICollectionView, numberOfItemsInSection section: Int) -> Int {
        return data.count
    }

    func collectionView(_ collectionView: UICollectionView, cellForItemAt indexPath: IndexPath) -> UICollectionViewCell {
        let cell = collectionView.dequeueReusableCell(withReuseIdentifier: "cell", for: indexPath)
        cell.contentView.backgroundColor = .blue
        let label = UILabel(frame: cell.contentView.bounds)
        label.text = data[indexPath.row]
        label.textAlignment = .center
        cell.contentView.addSubview(label)
        return cell
    }
}
```

In SwiftUI, the process is much simpler. Here’s how you can achieve the same grid layout with `LazyVGrid`:

```swift
import SwiftUI

struct ContentView: View {
    let data = ["Item 1", "Item 2", "Item 3", "Item 4"]
    
    var body: some View {
        ScrollView {
            LazyVStack(spacing: 20) {
                ForEach(data, id: \.self) { item in
                    Text(item)
                        .frame(width: 100, height: 100)
                        .background(Color.blue)
                        .cornerRadius(10)
                        .foregroundColor(.white)
                }
            }
            .padding()
        }
    }
}
```

With `ScrollView`, you can create smooth, scrollable interfaces that handle a large amount of content efficiently.


### 4. What is `List`?

`List` is another powerful way to create views in a loop. It is similar to `LazyVStack`, but with added functionality to handle data in a more structured and performant way. `List` is optimized for presenting a collection of data, and it automatically supports things like row selection, dynamic content, and efficient memory usage.

### How does it work?

You can use `List` to create a scrollable view of rows, where each row can display a dynamic piece of content from your data set. Just like `LazyVStack+ForEach`, `List` can take a data source and an identifier, and it will create a row for each item in the data source.

For example:

```swift
import SwiftUI

struct ContentView: View {
    let data = ["Item 1", "Item 2", "Item 3", "Item 4"]

    var body: some View {
        List(data, id: \.self) { item in
            Text(item)
                .padding()
        }
    }
}
```

### Customising List

`List` offers several built-in customizations for flexibility:

1. **Row Content**: Easily customize row content with `ForEach` or custom views inside the `List`.
2. **Row Styles**: Choose from built-in styles like `Plain`, `Grouped`, and `Inset` to match your app's design.
3. **Swipe Actions**: Add swipe actions like delete or edit without extra code.
4. **Row Editing**: Enable row editing for reordering, deleting, or inserting items.
5. **Section Headers/Footers**: Group rows with customizable headers and footers for better organization.
6. **Dynamic Changes**: Automatically handle data changes, with smooth updates and animations.
7. **Lazy Loading**: Use `List` in a `ScrollView` for lazy loading, optimizing performance with large datasets.

For example, we can add section headers to our `List` and apply an inset `.listStyle(.inset)'

```swift
List {
    Section(header: Text("Section 1")) {
        ForEach(1...5, id: \.self) { item in
            Text("Item \(item)")
        }
    }
    
    Section(header: Text("Section 2")) {
        ForEach(6...10, id: \.self) { item in
            Text("Item \(item)")
        }
    }
    .listStyle(.inset)
}
```

Since we're using sections in the `List`, we move the view creation loop into a `ForEach` within each section instead of directly in the `List` initializer. This allows us to group rows and apply headers effectively.


### What makes `List` different?

`List` differs from `LazyVStack+ForEach` by offering built-in support for scrolling and row selection, enabling interactions like taps and selections, as well as automatically providing swipe-to-delete functionality and editing modes for rows, making it easier to implement these common features without extra code. 

It also allows for sectioning, making it simple to group related items with section headers and footers. 

Like `LazyVStack+ForEach`, `List` optimises performance by only rendering the visible rows, improving memory usage and scrolling efficiency, and it provides customization for each row’s content and appearance.

### Deeper Dive

Under the hood, SwiftUI’s `List` is built on `UITableView`, inheriting many of its performance optimizations and features, including row reuse and efficient scrolling. This allows `List` to handle large datasets smoothly, just like `UITableView`.


### When to Use Each:

- Use `LazyVStack + ForEach` when you need customised layout or more control over individual views, such as animations, custom interactions, or unique styling. It is also ideal for performance when dealing with large data sets, especially if you need to implement your own selection, deletion, or sectioning behavior.
  
- Use `List` when you need built-in support for common list behaviors like row selection, swipe-to-delete, and section headers. `List` is the preferred choice for standard list views where you want to leverage SwiftUI's built-in optimizations and ease of use for displaying and interacting with dynamic data.

My personal opinion:

I always choose to go with `LazyVStack + ForEach` when creating looping views because it offers more flexibility in customising the views and layout.

## 4. View Lifecycle Events

In SwiftUI, you can manage lifecycle events such as when a view appears or when a task needs to be executed using modifiers like `.onAppear` and `.task`. These allow you to trigger specific actions when a view enters or exits the screen, or when a particular task should run asynchronously.

In UIKit equivalent, think `ViewDidLoad`.

#### `.onAppear`
The `.onAppear` modifier is called when the view becomes visible on the screen. It's useful for tasks like starting animations, fetching data, or performing any setup logic when a view appears.

```swift
Text("Hello, SwiftUI!")
    .onAppear {
        print("View has appeared!")
    }
```

Here, the closure inside `.onAppear` will be executed when the `Text` view first appears on the screen. It’s perfect for initiating actions that should only occur once when the view is presented.

#### `.task`
The `.task` modifier allows you to perform asynchronous tasks when a view appears. This is particularly useful for initiating data fetches or other background tasks when the view comes into focus.

```swift
Text("Loading Data...")
    .task {
        // Simulate fetching data
        let data = await fetchData()
        print(data)
    }
```

With `.task`, you can call an asynchronous function like `fetchData()` within the modifier, and SwiftUI will handle the task's lifecycle for you, ensuring the work is done when the view appears.


SwiftUI also automatically manages the lifecycle of the `.task`, ensuring that it's canceled if the view is no longer on the screen.

#### When to use each:

- Use `.onAppear` when you need to trigger actions that don’t require asynchronous operations (e.g., updating UI state, logging, or starting animations).
- Use `.task` when you need to perform asynchronous operations like data fetching or calling an API when the view appears and uou want to ensure that the task is performed only once when the view appears, even if there are multiple updates to the view.

## 5. Let's go through an example

Now that we've mastered using LazyVStack, LazyHStack, List ScrollViews and View lifecycle events, let's go through an example to demonstrate them.

