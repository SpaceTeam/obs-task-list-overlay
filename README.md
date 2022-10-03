# OBS Task List / Checklist Overlay

Largely left untouched, just some changed stylings to better fit our Streaming Setup

[![CI](https://github.com/geerlingguy/obs-task-list-overlay/actions/workflows/ci.yml/badge.svg?branch=master&event=push)](https://github.com/geerlingguy/obs-task-list-overlay/actions/workflows/ci.yml)

An HTML and Node.js-based task list overlay for OBS.

<img src="https://raw.githubusercontent.com/geerlingguy/obs-task-list-overlay/master/example.jpg" width="700" height="394" alt="OBS Task List Overlay Example with Jeff Geerling" />

I was frustrated with the lack of plugins that added a simple task list (with current task highlighted) to OBS, so I built this solution, which relies on OBS' 'Browser' source.

There are three components to this project:

  1. An HTML file (`index.html`), which lays out a stream title and task list/current status overlay.
  2. A Node.js HTTP server (`server.js`), which serves the HTML file to OBS, and allows you to control which step is highlighted in the display.
  3. A config file (`config.json`), which provides the server settings, title settings, and task list content.

I'm using this for a technical live stream, where I have a set list of milestones, and I want to indicate to viewers which task is currently being worked on.

To see it in practice, check out my [16 Drives, 1 Pi](https://www.youtube.com/watch?v=afnszOuWt74) livestream.

The design is inspired by the [NASASpaceflight](https://www.youtube.com/c/NASASpaceflightVideos) live stream status overlay, which I enjoy for it's simplicity and elegance.

## Customizing the overlay

All the styling for the overlay is embedded in the `index.html` file. If you want to tweak the appearance, it should be easy enough if you know basic CSS + JS.

To set a title and set up the list of tasks, copy the `example.config.json` file to `config.json`, and then edit the file to add in the settings you would like.

If you change the length of the stream title (`task_list_title`), you may also need to adjust the `task_list_title_width` to match the new width of the title text.

## Node.js App setup

After you add your own task list and title, you need to get the Node.js server app running.

First, [install Node.js](https://nodejs.org/en/download/) on your computer.

Next, run the following command in this directory to install the app's dependencies:

```
npm install
```

Then start the local server:

```
node server.js
```

You can open the overlay in a regular web browser by visiting `http://localhost:8080/`, but note that the element sizes and spacing will differ from what is output in OBS.

Always use OBS as the final reference for how things will look during a stream!

## Using Docker Compose

To use the Docker images configured with compose, you need [docker-compose](https://docs.docker.com/compose/install/). Once it's up a running, you simply do

```
docker-compose up --detach
```

To stop the server

```
docker-compose down
```

The config maps container's `8080` port to `localhost:8080`. If you need to use another localhost port, you need to change it in `docker-compose.yml` under the `ports` section. For example, to use the port `3000` you need to set the section as:

```yaml
    ports:
      - "3000:8080"
```

The port number after the colon would be the port configured in `config.json`.

## Adding the browser source in OBS

  1. In an OBS Scene, add a new 'Browser' Source.
  2. For the URL, enter `http://localhost:8080/` (use the port you have configured in `config.json`).
  3. Set the width to `1920` and height to `1080`.
  4. Check the 'Refresh browser when scene becomes active' option.
  5. Click 'OK', and the overlay should appear.

## Advancing to the next step

To control which task is highlighted, there are two paths the Node.js app responds to:

```
/up - Increase the current step by 1
/down - Decrease the current step by 1
```

You can also check what the current step is by requesting `/current`.

When you have the browser source open, OBS will change the highlighted step within one second of you requesting `localhost/up` or `localhost/down`.

The count can increase above the maximum number of items in your list, or below `1`, so you are responsible for making sure you don't go crazy calling `/up` or `/down` too many times :)

### Using an Elgato Stream Deck to advance steps

I use a Stream Deck to help with my live streaming, and I've created two buttons for this overlay, a 'back' (`/down`) and 'forward' (`/up`) key.

When I press the back button, the task list goes back one step.

When I press the forward button, the task list goes forward one step.

To add a hotkey in the 'Configure Stream Deck' app, drag a 'Website' button from the 'System' section into one of the slots on your Stream Deck.

Then set the URL to `http://localhost/up` (or `/down`), and check 'Access in background'.

Now, when you press that key, the appropriate URL is called silently and the on-screen task list should advance or go back a step depending on the button you pressed. Magic!
