# quick-xml

High performance xml pull reader/writer for simple enough xmls.

Inspired by [xml-rs](https://github.com/netvl/xml-rs).

[documentation](http://tafia.github.io/quick-xml/quick_xml/index.html)

## Usage

Carto.toml
```toml
[dependencies]
quick-xml="0.1"
```

``` rust
extern crate quick_xml;
```

## Example

### Reader

```rust
use quick_xml::{XmlReader, Event};

let xml = r#"<tag1 att1 = \"test\">
                <tag2><!--Test comment-->Test</b>
                <tag2>
                    Test 2
                </tag2>
            </tag1>"#;
let reader = XmlReader::from_str(xml).trim_text(true);
let mut count = 0;
let mut txt = Vec::new();
for r in reader {
    match r {
        Ok(Event::Start(ref e)) => {
            match e.as_bytes() {
                b"tag1" => println!("attributes values: {:?}", 
                                 e.attributes().map(|a| a.unwrap().1).collect::<Vec<_>>()),
                b"tag2" => count += 1,
                _ => (),
            }
        },
        Ok(Event::Text(e)) => txt.push(e.into_string()),
        Err(e) => println!("{:?}", e),
        _ => (),
    }
}
```

### Writer

```rust
use quick_xml::{Element, Event, XmlReader, XmlWriter};
use quick_xml::Event::*;
use std::io::Cursor;
use std::iter;

let xml = r#"<this_tag k1="v1" k2="v2"><child>text</child></this_tag>"#;
let reader = XmlReader::from_str(xml).trim_text(true);
let mut writer = XmlWriter::new(Cursor::new(Vec::new()));
for r in reader {
    match r {
        Ok(Event::Start(ref e)) if e.as_bytes() == b"this_tag" => {
            // collect existing attributes
            let mut attrs = e.attributes().map(|attr| attr.unwrap()).collect::<Vec<_>>();

            // adds a new my-key="some value" attribute
            attrs.push((b"my-key", "some value"));

            // writes the event to the writer
            assert!(writer.write(Start(Element::new("my_elem", attrs.into_iter()))).is_ok());
        },
        Ok(Event::End(ref e)) if e.as_bytes() == b"this_tag" => {
            assert!(writer.write(End(Element::new("my_elem", iter::empty::<(&str, &str)>()))).is_ok());
        },
        Ok(e) => assert!(writer.write(e).is_ok()),
        Err(e) => panic!("{:?}", e),
    }
}

let result = writer.into_inner().into_inner();
let expected = r#"<my_elem k1="v1" k2="v2" my-key="some value"><child>text</child></my_elem>"#;
assert_eq!(result, expected.as_bytes());
```

## Current state

**quick-xml** has been written to be fast.

On my first tests (200mb+ xmls) it performs much better (minimum 10x)
 than [xml-rs](https://github.com/netvl/xml-rs).

While this is a still WIP and only basic xml specifications are implemented, 
you can already use it for simple enough xmls (no namespaces, no exotic 
characters etc ...).

This is particularly true when the xml is generated by a known source (e.g. OpenStreetMap).

## Todo

There are many xml specifications not implemented yet:
- [ ] namespaces
- [ ] non-utf8
- [ ] parse xml prologue
- [ ] XQuries ?
- [ ] more checks
- [ ] ... and many other things I don't even know!

## Contribute

Any PR is welcomed!

I am not an expert in xml specifications, I simply have to work with big xmls. As a result, 
I may not implement some basic functionalities, simply because I don't know/need them.

## License

MIT
