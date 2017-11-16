## About the pattern

Facade pattern is one of the Structural Patterns. The main aim of it is to hide the complexity of system, class or logic and provide a simple interface  - use your system easier.
Commonly Facade is implemented in a way that one class is related to other classes which are representing a system logic. Please look at the diagram:

![diagram](./swift-facade-pattern/diagram.png)

As you can see, there is one class called `Facade` which separates the logic from `LogicA`, `LogicB`, `LogicC` classes. In the result of that, our client only call the `Facade` class in order to execute some methods that are implemented in other classes.

## Implementation

Let's imagine a simple scenario. You have already created a great app called `Super-Photo`. One of the core features of your app is saving/converting images with `JPEG` or `PNG` extension. In order to do this, you want to save `UIImage` representation in two ways. One is the saving it as `PNG` type, the second one is saving it as `JPEG` file type.

Firstly, you will need to create a two classes that will handle a photo saving with each extension:

```swift
class JPEGImageSaver {
    func save(image: UIImage, to path: URL, compressionQuality: CGFloat = 1.0) -> Bool {
        guard let imageData = UIImageJPEGRepresentation(image, compressionQuality) else { return false }
        do {
            try imageData.write(to: path)
            return true
        } catch {
            return false
        }
    }
}

class PNGImageSaver {
    func save(image: UIImage, to path: URL) -> Bool {
        guard let imageData = UIImagePNGRepresentation(image) else { return false }
        do {
            try imageData.write(to: path)
            return true
        } catch {
            return false
        }
    }
}
```

As you've noticed, our `JPEGImageSaver` takes `image`, `path` and `compressionQuality` parameters and saves the image as `JPEG` representation to a destinanation path.
`PNGImageSaver` do the same thing but it only takes `image` and `path` parameter.

Ok, so at the moment we have two classes with some logic inside. Now it's time to create a facade for it!

![diagram](./swift-facade-pattern/do_this.gif)

Create a class called `ImageSaverFacade` :

```swift
class ImageSaverFacade {
    private let pngImageSaver = PNGImageSaver()
    private let jpegImageSaver = JPEGImageSaver()

    init() {
    }

    @discardableResult
    func saveAsJPEG(image: UIImage, fileName: String, compressionQuality: CGFloat = 1.0) -> Bool {
        guard let destinationPath = createDestinationPath(fileName: fileName) else {
            return false
        }
        return jpegImageSaver.save(image: image, to: destinationPath, compressionQuality: compressionQuality)
    }

    @discardableResult
    func saveAsPNG(image: UIImage, fileName: String) -> Bool {
        guard let destinationPath = createDestinationPath(fileName: fileName) else {
            return false
        }
        return pngImageSaver.save(image: image, to: destinationPath)
    }

    private func createDestinationPath(fileName: String) -> URL? {
        guard let path = FileManager.default.urls(for: .documentDirectory, in: .userDomainMask).first else {
            print("Could not create destinationPath")
            return nil
        }
        return path.appendingPathComponent("\(fileName)")
    }
}
```

Our `ImageSaverFacade` class has two private objects of `PNGImageSaver` and `JPEGImageSaver` class. Because the client don't need to know anything about logic inside the only thing which `ImageSaverFacade` exposes to a public are two methods:

* `func saveAsPNG(image: UIImage, fileName: String) -> Bool`
* `func saveAsJPEG(image: UIImage, fileName: String, compressionQuality: CGFloat = 1.0)`

These are two things that our client should care about.

Now let's move on to the facade client part:

```swift
let image: UIImage = UIImage(named: "some_super_image")
let imageSaver = ImageSaverFacade()
imageSaver.saveAsPNG(image: image, fileName: "name_of_file")
imageSaver.saveAsJPEG(image: image, fileName: "name_of_file", compressionQuality: 0.5)
```

As you can see, it is super easy to save `UIImage` as `PNG` or `JPEG` using our `ImageSaverFacade`. Only thing to do is to choose the correct method.

## Conclusion

Facade design pattern can be used in many cases. Facade creates for you a simple gateway to a complicated system. Using Facade Pattern you will definitely make your code simpler to understand and read.



