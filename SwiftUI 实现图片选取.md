---
title: 如何用 SwiftUI 实现调用照片选取界面
date: 2021-09-30
tag: macOS
categories: iOS开发技术
top: false
mathjax: false
---

## 在 SwiftUI 中实现调用系统相册

### 0. 项目需求

许多 app 会要求用户上传照片到服务器或者 app 中进行后续处理，就会需要从设备相册中选取照片。这是一个相当普遍的需求。最终实现效果如下。

在 Xcode 预览模式下打开 live view 可以实时预览组件的运行情况，这里我们定义了一个选取照片按钮，而下方是一个 Image View，不过由于刚开始的时候没有选取任何照片，所以不显示任何内容。

![](https://pic.imgdb.cn/item/6154b5992ab3f51d91eec8eb.png)

当点击该按钮时，将会调出照片选取页面，也就是 Picker。

![](https://pic.imgdb.cn/item/6154b5a42ab3f51d91eecf9e.png)

这个 Picker 可以访问到设备（这里是 Simulator）的图库相册。由于 iOS 的使用习惯，大部分人会把大部分照片直接存在图库中，而不是存储到 “文件” 中，所以可以认为选取设备的所有照片。Picker 下方长条的按钮可以在 Picker 内在线预览所选中的照片。预览时支持放大、旋转等操作。

当选择完成后点击 “Add” 后，选取的照片加入到程序中，此时下方隐藏的 Image View 将显示选取的照片。

![](https://pic.imgdb.cn/item/6154b5b62ab3f51d91eedb83.png)

这就是这个组件所用到的功能。接下来介绍实现这个组件使用到的技术和实现方法。

### 1. 使用技术

#### 1.1 SwiftUI

SwiftUI 是苹果在2019年新推出的 Swift 原生 UI 框架。通过这个 UI 框架，开发者可以轻松实现美观大气的界面设计并且在全平台（iOS、iPadOS、macOS、tvOS、watchOS）上获得相同且美观的页面。比如，可以实现不同尺寸屏幕自适应布局，自动实现黑夜模式等等。同时，在开发过程中，Xcode 针对 SwiftUI 还提供了预览功能，可以实时显示界面设计效果，也可以在预览区域选取控件，左边的代码区域也会自动选中并自动修改属性。目前还在发展的阶段，不断有新的、更易用的 API 面世。这两年 Apple 在下一波大棋，把所有平台的软硬件都打通，这个 SwiftUI 也是实现这个愿景的一个工具。Apple 官方的应用和一些精品应用已经从 UIKit 转移到了 SwiftUI，这应该是一个趋势，就像 iOS 开发会从 object-C 迁移到 Swift 一样。

### 1.2 PhotoKit & PhotosUI

PhotoKit 是苹果近年来提出用于执行和照片相关功能的现代 API，PhotosUI 是其中的一个框架，可以实现比如图片选择等功能，下面是 Apple 对于 PhotoKit 的说明。

![](https://pic.imgdb.cn/item/6154bf9d2ab3f51d91f4a8c0.png)

PhotoKit 这几年功能不断完善，其中 PhotoUI 的各个组件由于美观的外观和异步执行、保护用户隐私的特性，逐步在取代原先 UIKit 中有关图片的组件，比如这篇文章介绍的 PHPickerViewController 就是 UIImagePickerViewController 的现代替代。

### 2. 代码

#### 2.1 PhotoPicker 图库选取器的编写

在 SwiftUI 中，与其他框架或语言常见的用类定义不同，是用结构体定义的，不得不说，Swift 语言的结构体的功能很强大。

```swift
struct PhotoPicker: UIViewControllerRepresentable {
  
}
```

这里要提到在 SwiftUI 中集合 UIKit 功能的方法，就是在实现结构体时要实现 `UIViewControllerRepresentable` 协议（这里的协议可以理解为类似于接口一样的东西），这个协议可以帮助 SwiftUI 实现一个 UIKit 中 UIViewController 组件的组件。

要实现协议，根据文档，我们就要实现如下方法和类。

![](https://pic.imgdb.cn/item/61551ff02ab3f51d913699b6.png)

![](https://pic.imgdb.cn/item/6155205c2ab3f51d91370c84.png)

下面就开始实现各个方法和类，不过，在此之前，要定义几个需要的成员变量。

```swift
		let configuration: PHPickerConfiguration
    @Binding var pickerResult: [UIImage]
    @Binding var isPresented: Bool
```

这里定义了三个变量，第一个是对 `PHPickerViewController` 的配置，第二个和第三个分别是选取结果和是否显示图库选取器界面。注意后两个变量前的修饰符 `@Binding`，代表这两个变量与外部的变量绑定，由 SwiftUI 管理，当外部变量更新时或内部变量发生变化时，可以实时显示变化。

接着实现协议中规定的两个方法。

```swift
		func makeUIViewController(context: Context) -> PHPickerViewController {
        let controller = PHPickerViewController(configuration: configuration)
        controller.delegate = context.coordinator
        return controller
    }
    
    func updateUIViewController(_ uiViewController: PHPickerViewController, context: Context) { }
```

第一个方法是绘制 `UIViewController` 组件，这里我们要绘制一个 `PHPickerViewController`，所以返回值是这个组件，方法体内创建了一个组件，并且将该组件的委托交给上下文的协调器，最后返回这个建立的组件。

第二个方法是更新 `UIViewController` 组件，这里我们不需要更新组件，就将方法体空着就好。

然后给用户提供一个协调对象。要实现 `makeCoordinator` 方法，并定义一个 Coordinator 类。请看如下代码：

```swift
		func makeCoordinator() -> Coordinator {
        Coordinator(self)
    }
    
    class Coordinator: PHPickerViewControllerDelegate {
        private let parent: PhotoPicker
        
        init(_ parent: PhotoPicker) {
            self.parent = parent
        }
        
        func picker(_ picker: PHPickerViewController, didFinishPicking results: [PHPickerResult]) {
            for image in results {
                if image.itemProvider.canLoadObject(ofClass: UIImage.self) {
                    image.itemProvider.loadObject(ofClass: UIImage.self) {
                        (newImage, error) in
                        if let error = error {
                            print(error.localizedDescription)
                        } else {
                            self.parent.pickerResult.append(newImage as! UIImage)
                        }
                    }
                } else {
                    print("Loaded Asset is not a Image")
                }
            }
            parent.isPresented = false
        }
    }
```

首先是 `makeCoordinator` 方法，创建一个协调器，并绑定调用的父类对象。这里要将协调的父类设为这个结构体（PhotoPicker）,所以要写为 `Coordinator(self)`。

然后我们要定义一个 `Coordinator` 类，这个类要负责和 ViewController 与其他 SwiftUI 协调。在委托模式中，这个类当然要设置为 `PHPickerViewControllerDelegate`。

这个类里有一个变量 `parent`，负责指定委托的父对象，这里的父对象是结构体 `PhotoPicker`，所以我们这样定义变量并且在初始化类时必须要传入一个有效的 `PhotoPicker` 对象。

在 Apple Developer 查询 `PHPickerViewControllerDelegate` 的文档，要实现 `picker` 方法去定义当用户在选择器在选择好以后或者点击取消（dismiss）后要执行的动作。通过查询文档，isPresented 变量是 SwiftUI 定义的一个变量，用来指示现在是否呈现拥有这个环境的组件是否呈现。

我们看 `picker` 方法。

```swift
		func picker(_ picker: PHPickerViewController, didFinishPicking results: [PHPickerResult]) {
            for image in results {
                if image.itemProvider.canLoadObject(ofClass: UIImage.self) {
                    image.itemProvider.loadObject(ofClass: UIImage.self) {
                        (newImage, error) in
                        if let error = error {
                            print(error.localizedDescription)
                        } else {
                            self.parent.pickerResult.append(newImage as! UIImage)
                        }
                    }
                } else {
                    print("Loaded Asset is not a Image")
                }
            }
            parent.isPresented = false
        }
    }
```

当点击成功或者取消时，都会返回一个 results，我们每次都需要处理这个 results，这里我们处理结果，并将结果中可读取的对象添加到 `pickerResult` 中。最后我们将父类的 `isPresented` 设置为 false，这样就能在执行完操作以后自动将图库选择器关闭。这里我们不需要对 UI 动画进行任何设置，所有动画都会由 SwiftUI 自动渲染。（不会有人觉得自己做动画能做过 Apple 吧...）

通过上边的代码，我们就成功定义了一个图库选择器。下面在 SwiftUI 中对该 `PhotoPicker` 进行测试。

#### 2.2 测试 PhotoPicker

写一个按钮测试 PhotoPicker 的功能。语法都是 SwiftUI 的基本语法，代码如下。

```swift
struct PhotoPickerDemo: View {
    @State private var isPresented: Bool = false
    @State var pickerResult: [UIImage] = []
    var config: PHPickerConfiguration {
        var config = PHPickerConfiguration(photoLibrary: PHPhotoLibrary.shared())
        config.filter = .images
        config.selectionLimit = 0
        return config
    }
    
    var body: some View {
        ScrollView {
            LazyVStack {
                Spacer()
                Button(action: { isPresented.toggle()}) {
                    Text("选取照片")
                        .font(.largeTitle)
                        .padding()
                        .background(Color.blue)
                        .foregroundColor(.white)
                        .cornerRadius(15)
                        .shadow(radius: 12)
                }
                .sheet(isPresented: $isPresented) {
                    PhotoPicker(configuration: self.config, pickerResult: $pickerResult, isPresented: $isPresented)
                }
                ForEach(pickerResult, id: \.self) {
                    image in
                    Image.init(uiImage: image)
                        .resizable()
                        .frame(alignment: .center)
                        .aspectRatio(contentMode: .fit)
                }
            }
        }
    }
}
```

就可以实现上面展示的效果了。

