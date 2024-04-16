<div class="page-head">
    <i>13/04/2024 ~ #testing ~ #foo ~ #bar</i>
    <h1><img src="assets/images/gameboy.png" title="GameBoy"> Test Chapter</h1>
    <p>The testing <span class="yellow">chapter</span></p>
</div>

Here is an inline example, $ \pi(\theta) $,
Look at this cool equation : $ \nabla f(x) \in \mathbb{R}^n, $

```rs
fn main() {
    println!("{}", hello());
}

/// A static string saying `Hello world!`
fn hello() -> &'static str {
    "Hello world!"
}
```

```cpp
class Scene {
  /* Graphic scene data and state */ 
  GraphicScene graphics;
  /* Physical scene data and state */ 
  PhysicalScene physics;
};

class CrateProp {
  CrateProp(Scene& scene) {
    /* "shadow_sprite" is optional, it is used for determining what part of the sprite casts shadows */
    /* if none is supplied the sprite will cast no shadows */
    this->sprite = scene.graphics.add_sprite("path/to/sprite", "path/to/shadow_sprite");
    /*! when the sprite goes out of scope, it is automatically cleaned up */

    /* the rigid body is automatically updated, this is the responsibility of the PhysicalScene */ 
    this->body = scene.physics.add_rb(/* pos */vec2(0, 1), /* size */vec2(1, 1));
    /*! when the rigid body goes out of scope, it is automatically cleaned up */
  }
};
```
