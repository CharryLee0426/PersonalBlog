# Landmark: A SwiftUI Demo Project

## 0. Review

**Landmark** is a SwiftUI Demo Project that shows a series of beatiful views in the US. It shows varies of features of SwiftUI and iOS APIs. Because of incredible capacity of the Swift Playground, now you can write your app wherever your iPad or Mac.

### 1. The Structure of this App

The structure of the App is below:

```
---App.swift
|
---Model/
|
---Views/
|
---Assets/
|
---Resources/
```

Model contains the data types and methods for obtaining the data from the folder resources. Views contains the view layout and widgets you designed. Assets and Resources are both the meterial that the app used. Exactly, Assets in Swift Playground stores the image files and text files such as JSON files are in the Resources folder.

## 2. Model

Model is the foundation of this app. Model, or Data in other words, is the foundation of the app. The app is a tool to get, produce and manage data for users in fact. In Model folder, there are two swift files, `Landmark.swift` and `ModelData.swift`.

`Landmark.swift` is a defination of the data type, Landmark. In this swift file, we defined a struct called `Landmark` to read the JSON file and covert some property to useful member, such as `coordinate` and `image`.