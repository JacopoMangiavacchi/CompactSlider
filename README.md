<p align="center">
  <img width="640" alt="cover3" src="https://user-images.githubusercontent.com/284922/166153877-97536d02-1feb-4018-961a-c3646faffdc0.png">
</p>
<p align="center">
  <img src="https://img.shields.io/badge/Swift-5.6-orange" />
  <img src="https://img.shields.io/badge/SwiftUI-2-blue" />
  <img src="https://img.shields.io/badge/macOS-11-lightgrey" />
  <img src="https://img.shields.io/badge/iOS-14-blue" />
  <img src="https://img.shields.io/badge/watchOS-7-green" />
  <img src="https://img.shields.io/github/license/buh/CompactSlider" />
</p>

`CompactSlider` is a control for selecting a value from a bounded linear range of values.

The slider is a replacement for the build-in slider and is designed specifically for SwiftUI. For me, the main motivation for writing a component that already exists is the very slow performance under macOS (e.g. when you need to resize the screen with multiple sliders or when animating) and the severely outdated design. At the same time, I was inspired by the slider design that Apple's [Photos](https://www.apple.com/macos/photos/#edit-gallery) app developed, which makes heavy use of sliders.

- [Requirements](#requirements)
- [Installation](#installation)
- [Preview](#preview)
- Documentation
- [Usage](#usage)
  - [Single Value](#single-value)
  - [Range Values](#range-values)
- [Styling](#styling)
  - [Configuration](#configuration)
  - [Secondary Appearance](#secondary-appearance)
  - [Prominent Style](#prominent-style)
- [Advanced Layout](advanced-layout) and `CompactSliderState`
- [License](#license)

# Requirements

- Swift 5.6
- Xcode 13
- SwiftUI 2
- macOS 11
- iOS 14
- watchOS 7

*Some of the requirements could be reduced if there is a demand for them.*

# Installation 

1. In Xcode go to `File` ⟩ `Add Packages...`
2. Search for the link below and click `Add Package`
```
https://github.com/buh/CompactSlider.git
```
3. Select to which target you want to add it and select `Add Package`

# Preview

**macOS**

https://user-images.githubusercontent.com/284922/166230021-223e1ffb-75e2-41ab-9995-618ccb414f8a.mov

**iPadOS**

https://user-images.githubusercontent.com/284922/166307680-8dfc706f-9e25-4739-94da-1d655b640e56.mov

**iOS**

https://user-images.githubusercontent.com/284922/166308017-fab77043-80c7-4567-b096-7fae8ba05967.mov

**watchOS**

https://user-images.githubusercontent.com/284922/166314399-857a0612-1a47-4bf8-9454-48eb3b63d1ba.mov

# Usage

A slider consists of a handle that the user moves between two extremes of a linear “track”. The ends of the track represent the minimum and maximum possible values. As the user moves the handle, the slider updates its bound value.

## Single value

The following example shows a slider bound to the speed value in increments of 5. As the slider updates this value, a bound Text view shows the value updating.

![Speed](https://user-images.githubusercontent.com/284922/166335247-2351777f-7ef5-440d-af8e-410fd94ce3a2.gif)

```swift
@State private var speed = 50.0

var body: some View {
    CompactSlider(value: $speed, in: 0...100, step: 5) {
        Text("Speed")
        Spacer()
        Text("\(Int(speed))")
    }
}
```

When used by default, the range of possible values is 0.0...1.0:

```swift
@State private var value = 0.5

var body: some View {
    CompactSlider(value: $value) {
        Text("Value")
        Spacer()
        String(format: "%.2f", value)
    }
}
```

Using the `direction:` parameter you can set the direction in which the slider will indicate the selected value:

![Direction](https://user-images.githubusercontent.com/284922/166335936-3e4cfdac-eafa-42c6-8da7-4010751973d8.gif)

```swift
@State private var value = 0.5

var body: some View {
    CompactSlider(value: $value, direction: .center) {
        Text("Center")
        Spacer()
        String(format: "%.2f", value)
    }
}
```

## Range values

The slider allows you to retrieve a range of values. This is possible by initialising the slider with the parameters `from:` and `to:`. 

The following example asks for a range of working hours:

![Range Values](https://user-images.githubusercontent.com/284922/166336963-7ba1ebd8-80f1-401a-b13d-b365c30748c2.gif)

```swift
@State private var lowerValue: Double = 8
@State private var upperValue: Double = 17

var body: some View {
    HStack {
        Text("Working hours:")
        CompactSlider(from: $lowerValue, to: $upperValue, in: 6...20, step: 1) {
            Text("\(zeroLeadingHours(lowerValue)) — \(zeroLeadingHours(upperValue))")
            Spacer()
        }
    }
}

private func zeroLeadingHours(_ value: Double) -> String {
    let hours = Int(value) % 24
    return "\(hours < 10 ? "0" : "")\(hours):00"
}
```

# Styling

The slider supports changing appearance and behaviour. In addition to the standard style, the [Prominent style](#prominent-style) is also available.

To implement your own style, you need to implement the `CompactSliderStyle` protocol, which contains many parameters that allow you to define the view according to user events. The styles are implemented in the same pattern as [ButtonStyle](https://developer.apple.com/documentation/swiftui/buttonstyle).

## Configuration

`CompactSliderStyleConfiguration` properties:

<img width="640" alt="Configuration" src="https://user-images.githubusercontent.com/284922/166454886-b80d7a9e-928d-46f0-b1ec-a58af7f37ff4.png">

The following example shows how to create your own style and use the configuration:

```swift
public struct CustomCompactSliderStyle: CompactSliderStyle {
    public func makeBody(configuration: Configuration) -> some View {
        configuration.label
            .foregroundColor(
                configuration.isHovering || configuration.isDragging ? .orange : .black
            )
            .background(
                Color.orange.opacity(0.1)
            )
            .accentColor(.orange)
            .clipShape(RoundedRectangle(cornerRadius: 12))
    }
}

public extension CompactSliderStyle where Self == CustomCompactSliderStyle {
    static var `custom`: CustomCompactSliderStyle { CustomCompactSliderStyle() }
}
```

And now we can apply it:

```swift
@State private var value: Double = 0.5

var body: some View {
    CompactSlider(value: $value) {
        Text("Custom Style")
        Spacer()
        Text(String(format: "%.2f", value))
    }
    .compactSliderStyle(.custom)
}
```

<img width="640" alt="custom1@2x" src="https://user-images.githubusercontent.com/284922/167266441-473bdd18-e571-4dda-8128-09122a7117f4.png">

## Secondary Appearance

The slider consists of several secondary elements, which can also be defined within their own style or directly for the slider.

1. The `.compactSliderSecondaryColor` modifier allows you to set the color and opacity for the secondary slider elements. You can simply change the base color for secondary elements: `.compactSliderSecondaryColor(.orange)`.

2. Using the other signature of the modifier: `.compactSliderSecondaryColor', the color can be set individually for each secondary element.

3. Another modifier `.compactSliderSecondaryAppearance` gives you the ability to change the `ShapeStyle` for the progress view.

Let's take the previous example and change the secondary elements:

```swift
public struct CustomCompactSliderStyle: CompactSliderStyle {
    public func makeBody(configuration: Configuration) -> some View {
        configuration.label
            .foregroundColor(
                configuration.isHovering || configuration.isDragging ? .orange : .black
            )
            .background(
                Color.orange.opacity(0.1)
            )
            .accentColor(.orange)
            .compactSliderSecondaryColor(
                .orange,
                progressOpacity: 0.2,
                handleOpacity: 0.5,
                scaleOpacity: 1,
                secondaryScaleOpacity: 1
            )
            .clipShape(RoundedRectangle(cornerRadius: 12))
    }
}
```

<img width="640" alt="Custom Style 1" src="https://user-images.githubusercontent.com/284922/167267510-a7e312d3-e955-4cd7-a5bf-dfa9a77d93bc.png">

Now change the solid color of the progress view to a gradient using the `compactSliderSecondaryAppearance' modifier:

```swift
public struct CustomCompactSliderStyle: CompactSliderStyle {
    public func makeBody(configuration: Configuration) -> some View {
        configuration.label
            .foregroundColor(
                configuration.isHovering || configuration.isDragging ? .orange : .black
            )
            .background(
                Color.orange.opacity(0.1)
            )
            .accentColor(.orange)
            .compactSliderSecondaryAppearance(
                progressShapeStyle: LinearGradient(
                    colors: [.orange.opacity(0), .orange.opacity(0.5)],
                    startPoint: .leading,
                    endPoint: .trailing
                ),
                focusedProgressShapeStyle: LinearGradient(
                    colors: [.yellow.opacity(0.2), .orange.opacity(0.7)],
                    startPoint: .leading,
                    endPoint: .trailing
                ),
                handleColor: .orange,
                scaleColor: .orange,
                secondaryScaleColor: .orange
            )
            .clipShape(RoundedRectangle(cornerRadius: 12))
    }
}
```

<img width="640" alt="Custom Style 2" src="https://user-images.githubusercontent.com/284922/167268168-af0867fe-240f-41ee-a8bc-6afac7526c97.png">

One of the slider parameters allows you to control the visibility of the handle. For this gradient example, it can be disabled:

```swift
CompactSlider(value: $value, handleVisibility: .hidden) {
    Text("Custom Style")
    Spacer()
    Text(String(format: "%.2f", value))
}
.compactSliderStyle(.custom)
```

![Custom Style](https://user-images.githubusercontent.com/284922/167268544-f559fb2a-5350-4bf5-b8cc-133f962c06e8.gif)

## Prominent Style

Prominent style allows for a more dramatic response for the selected value.

![Prominent Style](https://user-images.githubusercontent.com/284922/166339707-76f966f3-c4dd-44b3-8f72-2a44f8d8de33.gif)

```swift
@State private var speed: Double = 0

var body: some View {
    HStack {
        Text("Speed:")
        CompactSlider(value: $speed, in: 0...180, step: 5) {}
            .compactSliderStyle(
                .prominent(
                    lowerColor: .green,
                    upperColor: .red,
                    useGradientBackground: true
                )
            )
        Text("\(Int(speed))")
    }
}
```

# Advanced Layout

# License

`CompactSlider` is available under the [MIT license](https://github.com/buh/CompactSlider/blob/main/LICENSE)


