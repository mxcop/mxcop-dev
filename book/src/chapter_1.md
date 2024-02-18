# Chapter 1

Here is an inline example, $ \pi(\theta) $,

an equation,

$$ \nabla f(x) \in \mathbb{R}^n, $$

and a regular \$ symbol.

```rust
fn main() {
    println!("{}", hello());
}

/// A static string saying `Hello world!`
fn hello() -> &'static str {
    "Hello world!"
}
```

> [!TIP]
> Optional information to help a user be more successful.

```rs
use body::MdDoc;
use head::MdHeader;
use chrono::Datelike;
use once_cell::sync::Lazy;
use std::str::from_utf8;
use tera::{Context, Tera};

pub mod body;
pub mod head;
mod syntect;

// Build the template files:
pub static TEMPLATES: Lazy<Tera> = Lazy::new(|| {
    let mut tera = Tera::default();

    if let Err(e) = tera.add_raw_template("post", include_str!("../tmpl/post.html")) {
        println!("Parsing error(s): {}", e);
        ::std::process::exit(1);
    };

    tera.autoescape_on(vec![]); // Disable auto escape.
    tera
});

pub struct Doc {
    pub meta: MdHeader,
    pub content: String,
}

pub fn render_post(filename: &str, doc: &str) -> Option<Doc> {
    let Some(doc) = MdDoc::from_file(filename, doc) else {
        return None;
    };

    // Setup the post context:
    let mut context = Context::new();
    context.insert("title", &doc.meta.title);

    let date = &doc.meta.date;
    let date_str = format!("{}/{}/{}", date.day(), date.month(), date.year());
    context.insert("date", &date_str);

    // Get the size + the extra size from the html.
    let size = doc.content.len() + 1930;
    context.insert("size", &size);

    context.insert("description", &doc.meta.desc);
    context.insert("article", &doc.content.trim());

    // Render the post.
    let content = TEMPLATES.render("post", &context).unwrap();
    let content = from_utf8(&minify_html::minify(
        content.as_bytes(),
        &minify_html::Cfg::default(),
    ))
    .expect("failed to minify post").to_string();

    let real_size = &(content.len() as f32 * 0.001);
    let real_size_str = format!("{:.2} kB", real_size);
    println!("{}\n- size: {}", filename, real_size_str);

    Some(Doc {
        meta: doc.meta,
        content,
    })
}
```
