+++
title = "SwiftUI Custom View Modifiers"
date = 2020-11-27

[taxonomies]
tags = ["swiftui", "swift"]
+++

SwiftUI에서 ViewModifier 를 이용하여 Custom View를 만드는 방법
<!-- more -->

## SwiftUI Custom View Modifiers 

### Create Custom Button
- 아래와 같은 Custom button view를 생성하려면 

<img width="167" alt="viewModifier1" src="https://user-images.githubusercontent.com/911787/100363789-91f93280-3040-11eb-9390-1932129bd642.png">

```swift
struct CustomButton: View {
    var body: some View {
        Button(action: {
            print("select button")
        }, label: {
            Text("Continue")
                .font(.system(size: 16))
                .foregroundColor(.white)
                .padding(.horizontal, 14)
                .padding(.vertical, 10)
                .background(Color.blue)
                .overlay(
                    RoundedRectangle(cornerRadius: 3)
                        .strokeBorder(style: StrokeStyle(lineWidth: 1))
                        .foregroundColor(Color(.sRGB, red: 0.1, green: 0.1, blue: 0.1, opacity: 1))
                        .cornerRadius(4)
                        .shadow(color: Color(.sRGB, red: 0, green: 0, blue: 0, opacity: 0.5), radius: 5, x: 0, y: 0)
                )
        })
    }
}
```

### Using Modifier
- 위와 동일한 결과를 아래와 같이 `ViewModifier` 를 통해서 구현할 수 있다.
```swift
struct CustomModifierButton: View {
    var body: some View {
        Button(action: {
            print("select button")
        }, label: {
            Text("Cancel")
                .modifier(ButtonModifier())
        })
    }
}


struct ButtonModifier: ViewModifier {
    func body(content: Content) -> some View {
        return content
            .font(.system(size: 16))
            .foregroundColor(.white)
            .padding(.horizontal, 14)
            .padding(.vertical, 10)
            .background(Color.green)
            .overlay(
                RoundedRectangle(cornerRadius: 3)
                    .strokeBorder(style: StrokeStyle(lineWidth: 1))
                    .foregroundColor(Color(.sRGB, red: 0.1, green: 0.1, blue: 0.1, opacity: 1))
                    .cornerRadius(4)
                    .shadow(color: Color(.sRGB, red: 0, green: 0, blue: 0, opacity: 0.5), radius: 5, x: 0, y: 0)
            )
    }
}

```

### Using Modifier added state parameter
<img width="268" alt="viewModifier2" src="https://user-images.githubusercontent.com/911787/100363802-97567d00-3040-11eb-84df-62714bbb086f.png">

- 만약 각각 다른 background color를 가진 버튼에 style을 `ViewModifier` 로 구현하고 싶다면 
- 아래와 같이 구현할수 있다.
```swift
struct ButtonModifier: ViewModifier {
    @State var backgroundColor = Color.red
	...
```
- 추가로 fontSize같은 value도 `@State` 변수로 추가할 수 있다.
- 전체 sample code

```swift
struct ModifierStackView: View {
    var body: some View {
        HStack {
            CustomBlueButton()
            CustomRedButton()
            Button(action: {
                print("select button")
            }, label: {
                Text("Cancel")
                    .modifier(ButtonModifier(backgroundColor: .green))
            })
        }
    }
}

struct CustomBlueButton: View {
    var body: some View {
        Button(action: {
            print("select button")
        }, label: {
            Text("Continue")
                .modifier(ButtonModifier(backgroundColor: .blue))
        })
    }
}

struct CustomRedButton: View {
    var body: some View {
        Button(action: {
            print("select button")
        }, label: {
            Text("Okay")
                .modifier(ButtonModifier(backgroundColor: .red))
        })
    }
}

struct ButtonModifier: ViewModifier {
    @State var backgroundColor = Color.red
    
    func body(content: Content) -> some View {
        return content
            .font(.system(size: 16))
            .foregroundColor(.white)
            .padding(.horizontal, 14)
            .padding(.vertical, 10)
            .background(backgroundColor)
            .overlay(
                RoundedRectangle(cornerRadius: 3)
                    .strokeBorder(style: StrokeStyle(lineWidth: 1))
                    .foregroundColor(Color(.sRGB, red: 0.1, green: 0.1, blue: 0.1, opacity: 1))
                    .cornerRadius(4)
                    .shadow(color: Color(.sRGB, red: 0, green: 0, blue: 0, opacity: 0.5), radius: 5, x: 0, y: 0)
            )
    }
}

```
