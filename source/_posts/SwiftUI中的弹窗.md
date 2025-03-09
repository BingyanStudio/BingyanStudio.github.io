---
title: SwiftUI中的弹窗
cover_image: /img/cover.png
abbrlink: 6998e65b
date: 2024-06-02 05:49:16
---

### 系统弹窗

在SwiftUI中，iOS官方贴心地为我们实现了多种弹窗形式，包括Alert、confirmationDialog，即系统弹窗。

#### Alert

Alert是一个简单的弹窗，用于向用户显示一条消息和一个或多个按钮，以便用户可以选择相应的操作。正如它的名字一样，Alert通常用于显示紧急或重要的消息。我们可以通过 .alert() 来创建Alert。举个例子：

```
struct ContentView: View         .alert(isPresented: $showAlert)     }}
```

Alert当点击按钮时，更改控制变量showAlert，弹出Alert。在警告框弹出的情况下，只能点击确认/取消按钮来退出当前视图。

Alert 的title、message和 dismissButton的样式（如颜色、字体）是由系统控制的，不可以直接通过修改Text视图的属性来改变。这是为了确保 Alert 与系统外观保持一致，例如light mode和dark mode的区别。

当有多个Alert时，可以用enum枚举显示不同的Alert：

```
struct ContentView: View     }     var body: some View             Button("显示第二个Alert")         }        // 不同的alertType对应不同的Alert        .alert(item: $alertType)         }    }}
```
#### confirmationDialog

confirmationDialog 是一个更为灵活的弹窗，它在用户执行某个操作时，用多个按钮提供不同的选择。在iPhone中会从屏幕底部滑出。

```
struct ContentView: View         .confirmationDialog("", isPresented: $showActionSheet, titleVisibility: .hidden)             Button("删除", role: .destructive) {}            Button("取消", role: .cancel) {}        }    }}
```

confirmationDialog### 自定义弹窗

在许多场景中，只有这样的系统弹窗实在是有点让人难以接受。所以一般会使用自定义的弹窗。

#### Overlay

Overlay 是一个视图修饰符，它可以用来在现有视图上层添加一个新的视图层。其大致结构如下：

```
struct ContentView: View         .overlay(            // 判断是否显示弹窗            showingPopup ? popupOverlayView : nil        )    }    // 弹窗的视图    var popupOverlayView: some View          // 为了模拟弹窗，给弹窗外的背景增加一个遮罩         .background(             Color.black.opacity(0.5)                 .edgesIgnoringSafeArea(.all)                 .onTapGesture                  }         )     }}
```
这时可以发现，Overlay和ZStack的意义几乎一样，都是在“垂直屏幕方向”增加视图，类似的也有相应的实现：

```
struct ContentView: View             // 要显示弹窗            if showingPopup                 // 第三层，弹窗内容，额外设置一个视图                PopupView(showingPopup: $showingPopup)                    .frame(width: 300, height: 200)                    .background(Color.white)                    .cornerRadius(10)                    .position(x: UIScreen.main.bounds.width / 2, y: UIScreen.main.bounds.height / 2)            }        }        .animation(.easeInOut, value: showingPopup)    }}struct PopupView: View         .onTapGesture         }    }}
```
#### sheet

sheet 是一个用来展示一个新视图的修饰符，这个新视图会覆盖在当前视图上。在视觉上，有“旧页面沉底，新页面弹出”的效果。因此sheet的页面是介于弹窗和单独页面之间的状态，与原页面有很强的逻辑关联。其适用情景也需要满足这一要求：

* 编辑表单或设置界面：当用户需要输入或编辑信息时，可以使用 sheet 显示一个包含表单或设置选项的视图。
* 显示详细信息：当用户点击列表中的项目或者卡片时，使用 sheet 来显示更详细的信息。
* 选择操作：当用户需要从一组选项中进行选择时，可以使用 sheet 来显示一个菜单或者操作列表。这种方式通常用于快速选择选项而无需导航到另一个页面。
如此一来，就可以确保用户在执行相关任务时保持上下文的关联性。其实现也是类似的：

```
struct ContentView: View         .sheet(isPresented: $showingSheet)     }}struct sheetView: View         // .presentationDetents([.fraction(0.3)])    }}
```
其中presentationDetents是一个很炫酷的功能，它可以控制sheet的展示范围。.fraction(0.3)表示只会弹出到屏幕的30%，如果多加几个数字，就会变成分段推出的效果。

上面的showingSheet是基于布尔值的控制，也可以用标识符控制，使用示例如下：

```
enum SheetType: Identifiable }struct ContentView: View             Button("Show Second Sheet")         }        .sheet(item: $activeSheet)         }    }}
```
### 总结

实际开发中，弹窗使用之广泛使得开发者常常会把弹窗封装起来，作为一个通用的组件，以方便反复使用。总而言之，弹窗是一个贴合app使用逻辑并广泛运用的场景，合理的弹窗设计和实现有重要的意义和价值。

