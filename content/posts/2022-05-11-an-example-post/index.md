+++
title = "An example post"
description = "Wubalubadubdub!"

[taxonomies]
tags = ["example"]
projects = ["phrame"]
+++

This is gonna be an example that would demonstrate a regular zola post, that contains a longer paragraph in the beginning, and then a "read more" tag, which would be used to split the post into two parts.

<!-- more -->

This is the rest of the post.


```rust,linenos,hl_lines=11 19-21
let foo == bar; ->

fn init_egl(
    surface: &wl_surface::WlSurface,
    width: u32,
    height: u32,
) -> (
    egl::context::NotCurrentContext,
    egl::surface::Surface<glutin::surface::WindowSurface>,
) {
    let mut display_handle = raw_window_handle::WaylandDisplayHandle::empty();
    display_handle.display = surface
        .backend()
        .upgrade()
        .expect("Connection has been closed")
        .display_ptr() as *mut _;
    let display_handle = raw_window_handle::RawDisplayHandle::Wayland(display_handle);

    let mut window_handle = raw_window_handle::WaylandWindowHandle::empty();
    window_handle.surface = surface.id().as_ptr() as *mut _;
    let window_handle = raw_window_handle::RawWindowHandle::Wayland(window_handle);

    // Initialize the EGL Wayland platform
    //
    // SAFETY: The connection is valid.
    let display = unsafe { egl::display::Display::new(display_handle) }
        .expect("Failed to initialize Wayland EGL platform");

    // Find a suitable config for the window.
    let config_template = glutin::config::ConfigTemplateBuilder::default()
        .compatible_with_native_window(window_handle)
        .with_surface_type(ConfigSurfaceTypes::WINDOW)
        .with_api(
            glutin::config::Api::GLES2 | glutin::config::Api::GLES3 | glutin::config::Api::OPENGL,
        )
        .build();
    let config = unsafe { display.find_configs(config_template) }
        .unwrap()
        .next()
        .expect("No available configs");

    let gl_attrs = glutin::context::ContextAttributesBuilder::default()
        .with_context_api(glutin::context::ContextApi::OpenGl(None))
        .build(Some(window_handle));
    let gles_attrs = glutin::context::ContextAttributesBuilder::default()
        .with_context_api(glutin::context::ContextApi::Gles(None))
        .build(Some(window_handle));

    // Create a context, trying OpenGL and then Gles.
    let context = unsafe { display.create_context(&config, &gl_attrs) }
        .or_else(|_| unsafe { display.create_context(&config, &gles_attrs) })
        .expect("Failed to create context");

    let surface_attrs = glutin::surface::SurfaceAttributesBuilder::<WindowSurface>::default()
        .build(
            window_handle,
            NonZeroU32::new(width).unwrap(),
            NonZeroU32::new(height).unwrap(),
        );
    let surface = unsafe { display.create_window_surface(&config, &surface_attrs) }
        .expect("Failed to create surface");

    (context, surface)
}
```