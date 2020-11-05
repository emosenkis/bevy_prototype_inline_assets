# Bundled assets for [Bevy](https://github.com/bevyengine/bevy)

## Features

- Uses assets from the local filesystem if present for easy development, falls
  back to bundled assets only for those that aren't found locally.
- Provides an easy to use `inline_assets!` macro for bundling assets (or the
  API can be used directly instead).

## Status

Works for the Bevy 0.3.0. See [Bevy issue
606](https://github.com/bevyengine/bevy/issues/606) and the linked PRs for
discussion of native Bevy support for bundling assets.

## Installation

``` shell
cargo add bevy_prototype_inline_assets
```

or, in your `Cargo.toml`:

``` toml
bevy_prototype_inline_assets = "0.1.0"
```

## Usage

``` rust
use bevy::asset::{AssetPlugin, LoadState};
use bevy::prelude::*;
use bevy_prototype_inline_assets::{inline_assets, InlineAssets, InlineAssetsPlugin};
use std::collections::HashMap;
use std::path::Path;

fn main() {
    let inline_assets = inline_assets![
        "assets/image.png",
        "assets/font.ttf",
        "assets/audio.mp3",
        ...
    ];
    App::build()
        .add_resource(inline_assets)
        .add_plugins_with(DefaultPlugins, |group| {
            group.add_after::<AssetPlugin, _>(InlineAssetsPlugin)
        })
        .init_resource::<HashMap<&'static Path, HandleUntyped>>()
        .add_startup_system(setup.system())
        .add_system(spawn.system())
        .run();
}

fn setup(
    mut commands: Commands,
    inline_assets: Res<InlineAssets>,
    asset_server: Res<AssetServer>,
    mut inline_asset_handles: ResMut<HashMap<&'static Path, HandleUntyped>>,
) {
    *inline_asset_handles = inline_assets.load_all(asset_server);
}

fn spawn(
    mut commands: Commands,
    inline_asset_handles: Res<HashMap<&'static Path, HandleUntyped>>,
    asset_server: Res<AssetServer>,
    loaded: Local<bool>,
) {
    if *loaded
        || asset_server.get_group_load_state(inline_asset_handles.values().map(|h| h.id))
            != LoadState::Loaded
    {
        return;
    }
    let handle =
        inline_asset_handles.get(Path::new("assets/image.png")).unwrap().clone().typed();
    ...
}
```
