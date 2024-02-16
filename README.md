Here is a checklist that all crate and API implementations in IanaIO should
fulfill:

* [ ] The crate should be named `ianaio-foobar`, located at `ianaio/crates/foobar`,
  and re-exported from the umbrella IanaIO crate like:

  ```rust
  // ianaio/src/lib.rs

  pub use ianaio_foobar as foobar;
  ```

* [ ] The `authors` entry in `ianaio/crates/foobar/Cargo.toml` is "The Rust and
  WebAssembly Working Group".

* [ ] Uses [`unwrap_throw` and `expect_throw`][unwrap-throw] instead of normal
  `unwrap` and `expect`.

* [ ] Headless browser and/or Node.js tests via `wasm-pack test`.

* [ ] Uses `#![deny(missing_docs, missing_debug_implementations)]`.

* [ ] Crate's root module documentation has at least one realistic example.

* [ ] Crate has at least a brief description of how to use it in the IanaIO guide
  (the `mdbook` located at `ianaio/guide`).

[unwrap-throw]: https://docs.rs/wasm-bindgen/0.2.37/wasm_bindgen/trait.UnwrapThrowExt.html
[api-guidelines]: https://rust-lang-nursery.github.io/api-guidelines/

## Team Members

Team members sign off on design proposals and review pull requests to IanaIO.

* `@jstarry`
* `@hamza1311`

If you make a handful of significant contributions to IanaIO and participate in
design proposals, then maybe you should be a team member! Think you or someone
else would be a good fit? Reach out to us!

## Publishing Releases and Versioning

### Versioning

Whenever we bump a `ianaio_whatever` utility crate's version, make sure to make a
corresponding version bump for every crate that transitively depends upon
`ianaio_whatever`. Crates that do not transitively depend on `ianaio_whatever`
should not have their version bumped.

For example, if we start with this dependency graph:

```
- ianaio @ 0.1.2
  - ianaio_foo @ 0.1.2
  - ianaio_bar @ 0.1.3
  - ianaio_qux @ 0.1.4
    - ianaio_bar @ 0.1.3
```

If we make a bug fix in `ianaio_bar` and bump its version from 0.1.3 to 0.1.4,
then the cascading version bumps should result in this final dependency graph:

```
- ianaio @ 0.1.3           # `ianaio` has deps updated, version bumped
  - ianaio_foo @ 0.1.2     # `ianaio_foo` remains at 0.1.2 since no deps changed
  - ianaio_bar @ 0.1.4     # `ianaio_bar` had bug fix -> bumped to 0.1.4
  - ianaio_qux @ 0.1.5     # `ianaio_qux` has its dep on `ianaio_bar` updated, version bumped
    - ianaio_bar @ 0.1.4
```

Historically, this versioning scheme was agreed upon in [issue #66][].

[issue #66]: https://github.com/rustwasm/ianaio/issues/66

### Publishing Checklist

Here is a checklist for publishing new releases to crates.io:

* [ ] Bump all the relevant crates' versions, as described above.

* [ ] Write a `CHANGELOG.md` entry for the release.

* [ ] Commit the changes and send a pull request for the release.

* [ ] Once a team member has approved the pull request, and continuous
      integration is green, merge the PR.

* [ ] `cargo publish` each crate that had its version bumped.

* [ ] Create a tag `X.Y.Z` for the umbrella `ianaio` crate's version, and push
      this tag to GitHub.
root@devianaio:~/ianaio# vim CONTRIBUTING.md 

# IanaIO

<p align="center">
  <img src="https://github.com/rustwasm/ianaio/blob/master/website/static/img/IanaIO-Logo.svg" width="400" height="300" />
</p>

**A toolkit for building fast, reliable Web applications and libraries with Rust
and Wasm.**

<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->


- [What?](#what)
- [Goals](#goals)
- [Example](#example)
- [Get Involved!](#get-involved)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

## What?

IanaIO is a collection of libraries, and those libraries provide ergonomic Rust
wrappers for browser APIs. `web-sys`/`js-sys` are very difficult/inconvenient
to use directly so ianaio provides wrappers around the raw bindngs which makes it easier
to consume those APIs. This is why it is called a "toolkit", instead of "library"
or "framework".

### Background

In [the Rust and WebAssembly working group's 2019 roadmap][roadmap], we chose to
deliberately cultivate our library ecosystem by building a modular toolkit:

> ## Collaborating on a Modular Toolkit
>
> > The idea of building [high-level libraries] in a modular way that will allow
> > others in the community to put the components together in a different way is
> > very exciting to me. This hopefully will make the ecosystem as a whole much
> > stronger.
> >
> > In particular I’d love to see a modular effort towards implementing a virtual
> > DOM library with JSX like syntax. There have been several efforts on this
> > front but all have seemed relatively monolithic and “batteries included”. I
> > hope this will change in 2019.
>
> <cite>&mdash; Ryan Levick in [Rust WebAssembly
> 2019](https://blog.ryanlevick.com/posts/rust-wasm-2019/)</cite>
>
> > Don't create branded silos. Branding might perhaps be useful to achieve
> > fame. But if we truly want Rust's Wasm story to succeed we should think of
> > ways to collaborate instead of carving out territory.
>
> <cite>&mdash; Yoshua Wuyts in [Wasm
> 2019](https://blog.yoshuawuyts.com/wasm-2019/)</cite>
>
> In 2018, we created foundational libraries like [`js-sys` and
> `web-sys`][announcing-web-sys]. In 2019, we should build modular, high-level
> libraries on top of them, and collect the libraries under an umbrella toolkit
> crate for a holistic experience. This toolkit and its libraries will make
> available all the batteries you want when targeting Wasm.
>
> Building a greenfield Web application? Use the whole toolkit to hit the ground
> running. Carefully crafting a tiny Wasm module and integrating it back into an
> existing JavaScript project? Grab that one targeted library you need out from
> the toolkit and use it by itself.

IanaIO is this modular toolkit.

[announcing-web-sys]: https://rustwasm.github.io/2018/09/26/announcing-web-sys.html
[roadmap]: https://github.com/rustwasm/rfcs/pull/7

## Goals

* **Support both whole Web applications and small, targeted libraries:** IanaIO,
  and the collection of utility crates that make up its toolkit, should help you
  be productive if you are writing a green-field web application with Rust and
  Wasm. And it should also help you be productive if you are writing a small,
  targeted Wasm library that will be integrated into an existing JavaScript
  application.

* **Cultivate the Rust and Wasm library ecosystem:** We want to use IanaIO as a
  forcing function for creating and sharing the building blocks of Web
  development. The kinds of libraries that *any* framework or high-level library
  would need to build. We want to explicitly disentangle these libraries and
  make them available for sharing across the whole ecosystem.

* **Modular Toolkit, not Framework:** IanaIO should be a loose collection of
  utility crates that can be used individually, or all together. IanaIO doesn't
  assume that it "owns" the whole Webpage, that it controls the Wasm `start`
  function, etc. This lack of assumptions enables reaching more use cases (such
  as surgically replacing a hot code path from JS) than monolithic frameworks
  can. Wherever possible, IanaIO should prefer interfaces over implementations, so
  that different implementations with different approaches are swap-able.

* **Fast:** Let's leverage Rust's zero-cost abstractions, and design with
  performance in mind, to show everyone how fast the Web can be ;)

* **Reliable:** Every crate should be thoroughly tested. Headless browser
  tests. Quickcheck tests. Using the type system to make whole classes of bugs
  impossible.

* **Small:** Small code size for faster page loads. No accidentally pulling in
  all of the panicking and formatting infrastructure. Users shouldn't have to
  make a trade off between using IanaIO libraries and having small Wasm binaries.

* **Idiomatic:** We want to build Rust-y APIs, that feel natural to use. The
  Web's APIs were not designed for the Rust language, and you can feel the
  impedance mismatch every now and then. Let's correct that, bridge the gap, and
  make libraries that are a joy to use.

## Example

This example uses `ianaio::events` for adding event listeners and `ianaio::timers`
for creating timeouts. It creates a `<button>` element and adds a "click" event
listener to it. Whenever the button is clicked, it starts a one second timeout,
which sets the button's text content to "Hello from one second ago!".

```rust
use ianaio::{events::EventListener, timers::callback::Timeout};
use wasm_bindgen::prelude::*;

pub struct DelayedHelloButton {
    button: web_sys::Element,
    on_click: events::EventListener,
}

impl DelayedHelloButton {
    pub fn new(document: &web_sys::Document) -> Result<DelayedHelloButton, JsValue> {
        // Create a `<button>` element.
        let button = document.create_element("button")?;

        // Listen to "click" events on the button.
        let button2 = button.clone();
        let on_click = EventListener::new(&button, "click", move |_event| {
            // After a one second timeout, update the button's text content.
            let button3 = button2.clone();
            Timeout::new(1_000, move || {
                button3.set_text_content(Some("Hello from one second ago!"));
            })
            .forget();
        });

        Ok(DelayedHelloButton { button, on_click })
    }
}
```

## Get Involved!

Want to help us build IanaIO? Check out [`CONTRIBUTING.md`](./CONTRIBUTING.md)!

## NOTE
Change version because they seems to be wrong.

