<!--
SPDX-FileCopyrightText: 2026 GoJean
SPDX-License-Identifier: Apache-2.0
-->

# vulkan-rs: Rust bindings for Vulkan

- [Overview](#overview)
- [CI/CD status](#cicd-status)
- [Documentation](#documentation)
- [Installation and usage](#installation-and-usage)
  - [Cargo](#cargo)
  - [Vulkan SDK](#vulkan-sdk)
- [Breaking Changes](#breaking-changes)
  - [v0.1.0](#v010)
- [Contributing](#contributing)
- [License](#license)

## Overview

vulkan-rs provides safe, idiomatic Rust bindings for the Vulkan C API, modeled closely after the design of [Vulkan-Hpp](https://github.com/KhronosGroup/Vulkan-Hpp).

The goal is to offer a developer experience similar to Vulkan-Hpp — type-safe enumerations and bitfields, `Result`-based error handling, builder-style struct initialization, container support, and RAII-capable handle wrappers — while embracing Rust's ownership and safety guarantees.

Where Vulkan-Hpp leverages C++ features such as exceptions, templates, and move semantics, vulkan-rs maps the same concepts onto Rust idioms:

- Enumerations and bitflags become newtype wrappers with associated constants, plus `BitOr`/`BitAnd` and other operator implementations.
- Vulkan structs gain builder-style `Default`-based initialization and chainable `push_next` for `pNext` extension structures.
- Fallible Vulkan calls return `Result<T, vk::Result>`, with dedicated result-value types for functions that return data alongside a result code.
- Handles come in owning (`UniqueHandle`-style, akin to `std::unique_ptr`), shared (`SharedHandle`-style, akin to `std::shared_ptr`), and `raii` flavours, the latter providing object-oriented, automatically destroyed wrappers analogous to `vk::raii` in Vulkan-Hpp.

This repository contains the generator for vulkan-rs, which accepts the [XML specification of Vulkan](https://raw.githubusercontent.com/KhronosGroup/Vulkan-Docs/refs/heads/main/xml/vk.xml) and emits the Rust bindings documented further below.

## CI/CD status

We test across stable, beta, and nightly Rust toolchains on Windows, Linux, and MacOS (via [MoltenVK](https://github.com/KhronosGroup/MoltenVK)).

Below is a matrix representing the runner permutations used for automated testing.

For more details, check out the [workflow](.github/workflows/build.yml) that governs these runners.

| [![CI Status](https://github.com/GoJean/vulkan-rs/actions/workflows/build.yml/badge.svg)](.github/workflows/build.yml) | stable | beta | nightly |
| :--- | :---: | :---: | :---: |
| **Windows 2022** <br> **Windows 2025**            | ✓ <br> ✓ | ✓ <br> ✓ | ✓ <br> ✓ |
| **Ubuntu 22.04** <br> **Ubuntu 24.04**            | ✓ <br> ✓ | ✓ <br> ✓ | ✓ <br> ✓ |
| **MacOS 14** <br> **MacOS 15** <br> **MacOS 26**  | ✓ <br> ✓ <br> ✓ | ✓ <br> ✓ <br> ✓ | ✓ <br> ✓ <br> ✓ |

## Documentation

All other documentation is in [`docs`](./docs/):

1. [Building](./docs/Building.md) describes how to configure, build, and regenerate the bindings.
2. [Usage](./docs/Usage.md) explains how to use the various features of vulkan-rs with detailed examples.
3. [Configuration](./docs/Configuration.md) lists all the options available to configure the behaviour and features of vulkan-rs, including cargo features for selecting Vulkan versions and extensions.
4. [Handles](./docs/Handles.md) is an overview of the three different families of handles provided by vulkan-rs.

   These offer semantics similar to `std::unique_ptr`, `std::shared_ptr`, and `raii`-style automatic resource management, mirroring the `vk::raii` types from Vulkan-Hpp.

> [!NOTE]
>
> All macro configuration options [available in the C API](https://docs.vulkan.org/spec/latest/appendices/boilerplate.html) are also available and respected by vulkan-rs.

## Installation and usage

### Cargo

Add vulkan-rs to your `Cargo.toml`:

```toml
[dependencies]
vulkan-rs = "0.1"
```

Enable the Vulkan version and extensions you need via cargo features:

```toml
[dependencies]
vulkan-rs = { version = "0.1", features = ["vk1_3", "ext_descriptor_indexing"] }
```

For the latest unreleased changes, you can depend on the git repository directly:

```bash
cargo add vulkan-rs --git https://github.com/GoJean/vulkan-rs
```

A basic usage sketch, mirroring the Vulkan-Hpp `vk::raii` style:

```rust
use vulkan_rs::raii;

let entry = vulkan_rs::Entry::linked();
let instance = entry.create_instance(&vk::InstanceCreateInfo::default(), None)?;
let physical_devices = instance.enumerate_physical_devices()?;
```

### Vulkan SDK

Like Vulkan-Hpp, vulkan-rs relies on the Vulkan headers (and optionally the Vulkan loader) provided by the [LunarG Vulkan SDK](https://www.lunarg.com/vulkan-sdk/).

The recommended installation method is to install the SDK, which supplies both the XML specification used by the generator and the loader libraries used at run time.

If you need a more recent headers version than the SDK supports, vulkan-rs can also be pointed at a checkout of the [Khronos Group Vulkan-Headers repository](https://github.com/KhronosGroup/Vulkan-Headers).

## Breaking Changes

We try to keep the API for all flavours of vulkan-rs constant or backwards compatible.

However, we may introduce unavoidable breaking changes, usually to fix bugs or to track upstream Vulkan-Hpp API changes.

Following is a list of those changes, arranged by version.

### v0.1.0

Initial release.

The API surface is modeled after Vulkan-Hpp v1.4.x, including the following alignments:

- `raii::Device::acquire_next_image2_khr` and `raii::SwapchainKHR::acquire_next_image` return a result-value type rather than a plain tuple, matching the change in Vulkan-Hpp v1.4.324.
- Functions taking C-arrays of values (such as `set_blend_constants` and `set_fragment_shading_rate_khr`) take fixed-size arrays instead, matching Vulkan-Hpp v1.4.351.
- The named module equivalent, `vulkan_rs::prelude`, is the recommended import.

## Contributing

Feel free to submit a PR to add to this list.

You should run `rustfmt` and `clippy` (with warnings as errors) before submitting generated files.

- [Vulkan-Hpp](https://github.com/KhronosGroup/Vulkan-Hpp) The C++ bindings this project is modeled after — an invaluable reference for API design.

## License

Copyright 2026 GoJean

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.

This project is inspired by and models its API after
[Vulkan-Hpp](https://github.com/KhronosGroup/Vulkan-Hpp),
Copyright 2015-2026 The Khronos Group Inc., licensed under the Apache License, Version 2.0.