# Plugins

One of the most important features behind the engine is the ability to develop a **plugin**. 

Plugins have the ability to talk to other plugins, implement game logic and game behavior.

# How to develop a plugin

Firstly, plugins can be made entirely independant of the engine in their own repositories or folders.

To get started, clone the plugin template repo like so:

## Method 1: [GitHub Cli](https://cli.github.com/)
```sh
gh repo create <my-project-name> --public --template ChatImproVR/template

gh repo clone <Name>/<my-project-name>
```

## Method 2: Terminal
```sh
git clone git@github.com:ChatImproVR/template.git && \
    cd template && \
    git remote rm upstream
```

After you clone this repository you can begin working on your plugin!

# Important Plugin Functions
> WIP
