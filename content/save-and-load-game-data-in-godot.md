+++
title = "Save and load game data in Godot"
date = 2020-05-29

[taxonomies]
tags = ["gamedev", "godot"]
+++

In this tutorial, we will create simple save and load
functionality for our [Godot](https://godotengine.org) game.

Implementing this feature early on in the game development
cycle can save you a lot of pain further down the line
it also forces you to think about how data is structured
and managed in your game. All good things to be thinking
about.

I'll be using Godot 3.2.1 for this tutorial and writing
the code in GDScript. We aren't using any features
in the engine that are likely to change, so if you're on
an earlier or later version the code should still work.
I've been using the same code in my games since version
2, without any changes.

We will be structuring our game data using a GDScript
dictionary and serializing it to JSON. We will then read
the JSON file back and load it into our game data.

# Creating a new project

Fire up Godot and create a new project.

<img class="img-responsive" src="/godot/save_and_load/new_project_godot.png"
alt="New Godot Project" />

The first thing is to create a 'Root Node', for this we will select
`Other Node`, click `Node` and call it `Main`.

<img class="img-responsive" src="/images/godot/save_and_load/create_main_node.png"
alt="Add Main Node" />

To make our game data accessible, we will need to
create a script that is autoloaded on startup, I tend to name
this `global.gd`.

Create a folder in the resources tab called `Scripts` and then
create a new Godot script file in that folder by right-clicking
and selecting `New Script...`

<img class="img-responsive" src="/images/godot/save_and_load/new_script_global.png"
alt="Create Singleton Godot Script" />

# Creating our game data

Open the `global.gd` script and add some dummy game data. This will
be a [Dictionary](https://docs.godotengine.org/en/stable/classes/class_dictionary.html)
type and holds key/value pairs.

```
extends Node

var game_data = {
	"player": {
		"name": "Super Mario",
		"coins": 10
	}
}

```

We create a simple player key, which will hold our player data, in this
case, a name and a number of coins.

Next, we will need to make this file autoload so that it can be accessed
anywhere from within our game. Click `Project -> Project Settings` and
select the `AutoLoad` tab. Click the folder icon next to `Path` and select
the `global.gd` file from our scripts folder.

The node name will pre-populate with `Global`. Click the `Add` button to
add it and then click `Close`.

<img class="img-responsive" src="/images/godot/save_and_load/autoload_plugin.png"
alt="Create Singleton Godot Script" />

We just created a singleton node, only one instance of this will exist
during runtime, so if a script changes the data, it is changed for all.
With this approach we can show the coin count on the screen and as soon
as a coin is collected and the data changes, the label will update
automatically.

# Save Function

Lets create a new function called `save_game` in our globals script.

```
func save_game():
	var file = File.new()
	var location = "user://save_game.sav"

	print("Saving game to: " + OS.get_user_data_dir())
	if file.open(location, File.WRITE) != 0:
		print("Cannot write save file to " + location)
	else:
		file.store_line(to_json(game_data))
		file.close()
```

In this function, we create a new `File` object, assign a location to
save the file. We then check if we can write to the location that we
assigned, printing out an error if not. If the location is writeable
we serialize our `game_data` into JSON and write it to the file using
the `store_line` method. Finally, we close the file.

The location of the `user://` path differs depending on the platform.
For Windows, this usually exists within `APPDATA` and for Linux/Mac it
is `~/.PROJECT_NAME`. I added a `print` statement so that the location
is outputted to the console on save.

# Load Function

The load function will open our save game file and populate the data
back into a GDScript Dictionary object.

Let's create a new function called `load_save_file`.

```
func load_save_file():
	var file = File.new()
	file.open("user://save_gam.sav", file.READ)
	var text = file.get_as_text()
	game_data = parse_json(text)
	file.close()
```

We again create a new `File` object and open the save game file from
its location. We read the text from the file and parse the JSON,
storing it back in our game_data dictionary. Finally, we close the file.

<img class="img-responsive" src="/images/godot/save_and_load/save_load_script.png"
alt="Save and Load game script for Godot" />

# Rendering our game data

Let's visualise this a little more, by adding a couple of labels and buttons.
Add two labels to the `Main` scene and name them `PlayerName` and `PlayerCoins`.
You can add some dummy text, we will be setting this when the game loads.

Add a script to the `Main` root node we created earlier. Let's change
the `_process` function, so that it sets the player name and coin labels.
This function is called every frame, so when we later modify our save game
and load it back in, we will see the labels update.

```
func _process(delta):
	$PlayerName.text = Global.game_data.player.name
	$PlayerCoins.text = "Coins: " + str(Global.game_data.player.coins)
```

You will notice that we set the label text with our game data, that is
accessible through the singleton node `Global` that we created.

The `str` function is casting our coins variable to a string so that it
can be appended to the `Coins:` string. Without it, you will get a type
error. We could have stored our coins as a string, but I think it's
good practice to be strict with your types.

# Create save and load buttons

Add two buttons to your `Main` node and name them `SaveGame` and `LoadGame`.

<img class="img-responsive" src="/images/godot/save_and_load/ui_layout.png"
alt="Save and Load buttons" />

We will connect a `button_up` signal for each button that connects to our
`Main` node. This will automatically create two functions within our `Main`
script.

Click the `Node` tab and select the `button_up` signal. Select `Main` and
click the `Connect` button. Do this for both buttons.

<img class="img-responsive" src="/images/godot/save_and_load/connect_save_game_signal.png"
alt="Connect Godot Signal to Node" />

You should now have two new functions in `Main`. Let's now call our save
and load functions when the respective buttons are clicked.

```
func _on_SaveGame_button_up():
	Global.save_game()

func _on_LoadGame_button_up():
	Global.load_save_file()
```

If you run your game and click the `Save Game` button you should see the
location of the same game file printed out to the console.

```
--- Debugging process started ---
Godot Engine v3.2.1.stable.custom_build - https://godotengine.org
OpenGL ES 3.0 Renderer: GeForce GTX 1060 6GB/PCIe/SSE2

Saving game to: /home/richard/.local/share/godot/app_userdata/SaveAndLoadGame
```

Whilst the game is still running, open up the save game file from the
location that was printed out. Edit the coins value and save, then go back
to the game and click `Load Game`. You should see the coins value automatically
update!

<img class="img-responsive" src="/images/godot/save_and_load/LoadGameExample.gif"
alt="Example of save and load game in Godot" />

# Final notes

This approach is very flexible and allows you to better control your game data.
There are some caveats, however. The Godot dictionary data is held in memory, this
usually isn't a problem, but if you have a game where you're procedurally generating
a lot of data, you're going to hit some performance issues. I previously worked
on a prototype that procedurally generated a city with over a hundred thousand
citizens. This occupied a huge amount of memory and I ended up having to write
a native extension to handle this, as opposed to using the above approach.

I would also separate the game data from other data that is static, such as
weapon names, inventory items and gear. They don't need to be stored in the save
file. Thinking about this separation of data early on in the development cycle
is good and will save you from a lot of pain further down the line.
