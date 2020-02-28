[![Docs.rs](https://docs.rs/com/badge.svg)](https://docs.rs/crate/show-image/)
[![CI](https://github.com/robohouse-delft/show-image-rs/workflows/CI/badge.svg)](https://github.com/robohouse-delft/show-image-rs/actions?query=workflow%3ACI+branch%3Amaster)

# show-image

`show-image` is a library for quickly displaying images.
It is intended as a debugging aid for writing image processing code.
The library is not intended for making full-featured GUIs,
but you can process keyboard events from the created windows.

## Supported image types.
The library aims to support as many different data types used to represent images.
To keep the dependency graph as small as possible,
support for third party libraries must be enabled explicitly with feature flags.

Currently, the following types are supported:
  * Tuples of binary data and `ImageInfo`.
  * `image::DynamicImage` and `image::ImageBuffer` with the `image` feature.
  * `tch::Tensor` with the `tch` feature.

If you think support for a some data type is missing,
feel free to send a PR or create an issue on GitHub.

## Event handling.
You can receive events using `Window::events`.
This is a general channel on which all events for that window are sent.

You can also handle keyboard events for windows using `Window::wait_key` or `Window::wait_key_deadline`.
These functions will wait for key press events while discarding key up events.
Alternatively you can use `Window::add_key_handler` to register an asynchronous key handler.

## Saving displayed images.
If the `save` feature is enabled, windows allow the displayed image to be saved using `Ctrl+S`.
This will open a file dialog to save the currently displayed image.

Note that images are saved in a background thread.
To ensure that no data loss occurs, call `stop` to gracefully stop and join the background thread.

## Example 1: Showing an image.
This example uses a tuple of `(&[u8], ImageInfo)` as image,
but any type that implements `ImageData` will do.
```rust
use show_image::{ImageInfo, make_window};

let image = (pixel_data, ImageInfo::rgb8(1920, 1080));

// Create a window and display the image.
let window = make_window("image")?;
window.set_image(image, "image-001")?;

```

## Example 2: Handling keyboard events.
```rust
use show_image::{KeyCode, make_window};

// Create a window and display the image.
let window = make_window("image")?;
window.set_image(&image, "image-001")?;

// Print keyboard events until Escape is pressed, then exit.
// If the user closes the window, wait_key() will return an error and the loop also exits.
while let Ok(event) = window.wait_key(Duration::from_millis(100)) {
    if let Some(event) = event {
        println!("{:#?}", event);
        if event.key == KeyCode::Escape {
            break;
        }
    }
}

// Make sure all background tasks are stopped cleanly.
show_image::stop()?;
```
