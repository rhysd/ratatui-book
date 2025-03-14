# Full Async

One way to achieve full async behavior is to wrap the `App` struct in a `Arc<Mutex<App>>`.

The main `run` loop might look something like this:

```rust
  pub async fn run() -> Result<()> {
    let (action_tx, mut action_rx) = mpsc::unbounded_channel();

    let mut app = Arc::new(Mutex::new(App::new(action_tx.clone())));

    let mut tui = TerminalHandler::new(app.clone());

    loop {
      if let Some(action) = action_rx.recv().await {
        match action {
          Action::RenderTick => tui.render()?,
          Action::Quit => app.lock().await.quit(),
          action => {
            if let Some(_action) = app.lock().await.update(action) {
              action_tx.send(_action)?
            };
          },
        }
      }
      app.lock().await.should_quit {
        tui.stop()?;
        break;
      }
    }
    Ok(())
  }
```

And you might have a `tui.rs` file that looks like this:

```rust
pub struct TerminalHandler {
  pub task: JoinHandle<()>,
  tx: mpsc::UnboundedSender<Message>,
}

impl TerminalHandler {
  pub fn new(app: Arc<Mutex<App>>) -> Self {
    let (tx, mut rx) = mpsc::unbounded_channel::<Message>();

    let task = tokio::spawn(async move {
      let mut t = Tui::new().context(anyhow!("Unable to create terminal")).unwrap();
      t.enter().unwrap();
      loop {
        match rx.recv().await {
          Some(Message::Stop) => {
            t.exit().unwrap_or_default();
            break;
          },
          Some(Message::Suspend) => {
            t.suspend().unwrap_or_default();
            break;
          },
          Some(Message::Render) => {
            let mut _app = app.lock().await;
            t.draw(|f| {
              _app.render(f, f.size());
            })
            .unwrap();
          },
          None => {},
        }
      }
    });
    Self { task, tx }
  }

  pub fn suspend(&self) -> Result<()> {
    self.tx.send(Message::Suspend)?;
    Ok(())
  }

  pub fn stop(&self) -> Result<()> {
    self.tx.send(Message::Stop)?;
    Ok(())
  }

  pub fn render(&self) -> Result<()> {
    self.tx.send(Message::Render)?;
    Ok(())
  }
}
```

In this particular code above, since we take the lock to render, the `app` handle event or update
methods will not be called while rendering is occurring.

In order for this approach to be useful, you'll have to break your state down into different
structs. In cases where you do this, and have different parts of your app state being updated and
rendered, this approach may be viable. This is usually overkill and almost never required.
