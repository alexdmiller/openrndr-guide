# Writing Program extensions

## The extension interface

OPENRNDR provides a simple `Extension` interface with which default `Program` behaviour can be changed.

```kotlin
interface Extensions {
    fun setup(program:Program) {}
    fun beforeDraw(drawer:Drawer, program:Program) {}
    fun afterDraw(drawer:Drawer, program:Program) {}
}
```

In the `setup()` function the extension can setup itself and hook into program events.

The `beforeDraw()` function is called right before the program's `draw()` is executed.

The `afterDraw()` is called after the program's `draw()` is executed.

You can enabled and disable an extension by setting the `enabled` boolean on the extension.

## A simple extension

Presented here is the outline of a simple extension that overlays the frames per second on top of the program output.

```kotlin
class FPSDisplay:Extension {

    var frames = 0
    var startTime:Double

    fun setup(program:Program) {
        startTime = program.seconds
    }

    fun afterDraw(drawer:Drawer, program:Program) {
        frames++

        drawer.isolated {
            // -- set view and projections
            drawer.view = Matrix44.IDENTITY
            drawer.ortho()

            drawer.fontMap = [... some font]
            drawer.text("fps: ${frames/(program.seconds-startTime)}")
        }
    }
}
```

Using the `FPSDisplay` extension from your main program would then look like this:
```kotlin
class Main:Program {
    fun setup() {
        extend(FPSExtension())
    }
}
```

## Extension application order

In the scenario in which a program has 3 extensions installed like in the snippet below.

```
fun setup() {
    extend(ExtensionA())
    extend(ExtensionB())
    extend(ExtensionC())
}
```

This resulting application order of beforeDraw and afterDraw is then as follows:

```kotlin
    extensionA.beforeDraw()
    extensionB.beforeDraw()
    extensionC.beforeDraw()

    program.draw()

    extensionC.afterDraw()
    extensionB.afterDraw()
    extensionA.afterDraw()
```

As you can see, the afterDraw() calls are applied in reverse order, this order was decided on to help with push/pop order of transforms and styles.

# Debug Extensions

By default there are two debug extensions available that will provide view with the camera attached to the mouse. These extensions will help you navigate the render space and inspect objects both in 2D and 3D.

```kotlin
fun setup() {
    val extension = extend(Debug3D(Vector3(10.0, 10.0, 10.0), Vector3.ZERO, 40.0))

    // toggle the extension with the d-key
    keyboard.keyDown.filter { it.name == "d" }.listen {
        extension.enabled = !extension.enabled
    }
}
```
