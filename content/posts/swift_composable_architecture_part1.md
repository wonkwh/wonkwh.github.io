+++
title = "swift composable architecture part_1"
date = 2020-04-14

[taxonomies]
tags = ["swift", "tca", "swiftui", "combine", "pointfree"]
+++

# swift-composable-archtecture-part1

–	https://github.com/pointfreeco/swift-composable-architecture#installation
–	https://www.pointfree.co/collections/composable-architecture/a-tour-of-the-composable-architecture/ep100-a-tour-of-the-composable-architecture-part-1

## todo example
### 준비 작업
–	swift pacakage로 install
–	`http://github.com/pointfreeco/swift-composable-architecture` 추가 
–	state, action, environment 추가 후 Reducer 추가

```swift
import ComposableArchitecture
struct AppState {
        
}
enum AppAction {
    
}
struct AppEnvironment {
    
}
let appReducer = Reducer<AppState, AppAction, AppEnvironment> { state, action, _ in
    switch action {
    }
}

–	ContentView에 Store instance 추가 
struct ContentView: View {
    let store: Store<AppState, AppAction>
    
    var body: some View {
        NavigationView {
            List {
                Text("Hello, World!")
            }.navigationBarTitle("Todos")
        }
    }
}
```


–	ContentView_preview 와 sceneDelegate ContentView Constructor 변경
```swift
struct ContentView_Previews: PreviewProvider {
    static var previews: some View {
        ContentView(store: Store(
                initialState: AppState(),
                reducer: appReducer,
                environment: AppEnvironment()
            )
        )
    }
}
```
### ViewStore 추가
–	state에 domain data model을 추가 
```swift
struct ToDo: Equatable, Identifiable {
    var description = ""
    let id: UUID
    var isComplete = false
}
struct AppState: Equatable {
    var todos: [ToDo]
}
```

–	view 에 WithViewStore 을 추가 
```swift
    NavigationView {
        WithViewStore(self.store) { viewStore in
            List {
                ForEach(viewStore.todos) { todo in
                    Text(todo.description)
                }
            }.navigationBarTitle("Todos")
        }
    }
```
–	preview 에 dommy data를 AppState에  추가 
```swift
struct ContentView_Previews: PreviewProvider {
    static var previews: some View {
        ContentView(
            store: Store(
                initialState: AppState(todos: [
                    ToDo(description: "PointFree", id: UUID(), isComplete: false),
                    ToDo(description: "Udemy", id: UUID(), isComplete: false),
                    ToDo(description: "MoimApp", id: UUID(), isComplete: false),
                ]),
                reducer: appReducer,
                environment: AppEnvironment()
            )
        )
    }
}
```

### View UI 변경 
–	ListView에 체크마크 버튼을 추가하고 isComplete 속성에 따라 foregroundColor를 추가 
–	preview의 속성을 변경하면서 제대로 되는지 확인 할수 있다. 
```swift
    List {
        ForEach(viewStore.todos) { todo in
            HStack {
                Button(action: {}) {
                    Image(systemName: todo.isComplete ? "checkmark.square": "square")
                }
                TextField("Untitled todo", text: .constant(todo.description))
            }.foregroundColor(todo.isComplete ? .gray : nil)
        }
    }.navigationBarTitle("Todos")
```

### Action 추가 
- UI action을 `AppAction` enum 에 추가한다. 
```swift
    enum AppAction {
        case todoCheckboxTapped(index: Int)
        case todoTextFieldChanged(index: Int, text: String)
    }

```
- `Reducer` 에 해당 Action 구현
```swift
let appReducer = Reducer<AppState, AppAction, AppEnvironment> { state, action, _ in
    switch action {
        
    case .todoCheckboxTapped(index: let index):
        state.todos[index].isComplete.toggle()
        return .none
    case .todoTextFieldChanged(index: let index, text: let text):
        state.todos[index].description = text
        return .none
    }
}

```

- `ContentView`에 해당 Control Event에 Action send
    - Index를 Foreach에 추가하려면 다음과 같이 추가 
    - `ForEach(Array(viewStore.todos.enumerated()), id: \.element.id)
    - 추후 `ForEachStore` 로 변경

```swift
        ForEach(Array(viewStore.todos.enumerated()), id: \.element.id) { index, todo in
            HStack {
                Button(action: {
                    viewStore.send(.todoCheckboxTapped(index: index))
                }) {
                    Image(systemName: todo.isComplete ? "checkmark.square": "square")
                }.buttonStyle(PlainButtonStyle())

                TextField("Untitled todo", text: viewStore.binding(
                  get: { $0.todos[index].description },
                  send: { .todoTextFieldChanged(index: index, text: $0) }
                ))
            }.foregroundColor(todo.isComplete ? .gray : nil)
        }
```

### Debugging
- `Reducer` 에 `debug()` 추가
```swift
let appReducer = Reducer<AppState, AppAction, Void> { state, action, _ in
  ...
}
.debug()
```
- 혹은 `SceneDelegate` , `ContentView_Previews`에 contentView instance에 reducer에 reducer paramter에 debug()를 추가 
```swift
     let contentView = ContentView(store: Store(
                initialState: AppState(todos: [
                    ToDo(description: "PointFree", id: UUID(), isComplete: true),
                    ToDo(description: "Udemy", id: UUID(), isComplete: false),
                    ToDo(description: "MoimApp", id: UUID(), isComplete: false),
                ]),
                reducer: appReducer.debug(),
                environment: AppEnvironment()
            )
        )

```

- 결과는 다음과 같은 로그를 console에 뿌려준다. 

```
AppAction.todoCheckboxTapped(
    index: 1
  )
  AppState(
    todos: [
      ToDo(
        description: "PointFree",
        id: 2CA87ACF-3868-4B1F-9D95-D25C03FB6B02,
        isComplete: true
      ),
      ToDo(
        description: "Udemy",
        id: EB6738F4-B644-40B1-9519-8C7050FB55B3,
−       isComplete: false
+       isComplete: true
      ),
      ToDo(
        description: "MoimApp",
        id: 95E6E5BF-6FB1-430E-8444-B1B137369066,
        isComplete: false
      ),
    ]
  )
 ```
