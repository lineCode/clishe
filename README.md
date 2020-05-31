# Clishé

This is a thin wrapper around clap. It enables you to write your CLI using a
different idiom. This idiom is optimized for:

 - Reduction of boilerplate/plumbing for repetitive command declaration
 - Access to command args, mutable state (context), return type
 - Command implementation and definition live side-to-side

## Example

This is a simple app with dummy commands to display the spirit of clishe.

```rust
#[macro_use]
extern crate clap;
#[macro_use]
extern crate clishe;
use ::clishe::prelude::*;

fn main() {
    let mut ctx = Context("".to_owned());
    if let Err(err) = Food::parse().run(&mut ctx) {
        eprintln!("error: {}", err);
    } else {
        println!("success: {}", ctx.0);
    }
}

pub struct Context(String);

dispatchers! {
    Food(&self, _: &mut Context) -> Result<()> [
        Vegetables: vegetables::Vegetables,
        Meat: meat::Meat,
    ],
}

mod vegetables {
    use ::clishe::prelude::*;

    dispatchers! {
        #[clap(about = "Fresh from the garden")]
        Vegetables(&self, _: &mut crate::Context) -> Result<()> [
            Carrots: Carrots,
            Lettuce: Lettuce,
        ],
    }

    commands! {
        Carrots(&self, _ctx: &mut crate::Context) -> Result<()> {
            Ok(())
        } struct {
            name: String,
        }

        Lettuce(&self, ctx: &mut crate::Context) -> Result<()> {
            ctx.0 = if let Some(name) = self.name.as_ref() {
                format!("Hello, {}!", name)
            } else {
                "Hello, world!".to_owned()
            };

            Ok(())
        } struct {
            name: Option<String>,
        }
    }
}

mod meat {
    use ::clishe::prelude::*;

    dispatchers! {
        #[clap(about = "Nothing like a properly cooked meat")]
        Meat(&self, _: &mut crate::Context) -> Result<()> [
            Beef: Beef,
        ],
    }

    commands! {
        Beef(&self, ctx: &mut crate::Context) -> Result<()> {
            ctx.0 = format!("Hello, {}!", self.name);
            Ok(())
        } struct {
            name: String,
        }
    }
}
```

This code will provide you with the following program:

```
$ ./clishe
clishe

USAGE:
    clishe <SUBCOMMAND>

FLAGS:
    -h, --help       Prints help information
    -V, --version    Prints version information

SUBCOMMANDS:
    help          Prints this message or the help of the given subcommand(s)
    meat          Nothing like a properly cooked meat
    vegetables    Fresh from the garden

$ ./clishe vegetables lettuce friend
success: Hello, friend!
```

## TODO

### Bugs

### Features

 - Provide additional built-in shell using rustyline
 - Auto-completion integrated to the framework

### Testing

 - Test more use-cases

### Documentation

 - Example in the README.md does not show enough clap attributes used
