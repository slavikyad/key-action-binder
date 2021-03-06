# KeyActionBinder

KeyActionBinder tries to provide universal game input control for both keyboard and game controllers in Adobe AIR, independent of the game engine used or the hardware platform it is running in.

While Adobe Flash already provides all the means for using keyboard and game input (via [KeyboardEvent](http://help.adobe.com/en_US/FlashPlatform/reference/actionscript/3/flash/events/KeyboardEvent.html) and [GameInput](http://help.adobe.com/en_US/FlashPlatform/reference/actionscript/3/flash/ui/GameInput.html)), KeyActionBinder tries to abstract those classes behind a straightforward, higher-level interface. It is meant to be simple but powerful, while solving some of the most common pitfalls involved with player input in AS3 games.

## Using KeyActionBinder

### Basic setup

In KeyActionBinder, you evaluate your own arbitrary "actions" instead of specific keys or controls.

In your game setup block, create a `KeyActionBinder` instance.

	binder = new KeyActionBinder(stage);

### Keyboard bindings

You can then add actions with specific bindings to that instance. To add a keyboard binding that means pressing the left arrow key will activate a "move-left" action, and pressing the right arrow key will activate a "move-right" action, you do:

	binder.addKeyboardActionBinding("move-left", Keyboard.LEFT);
	binder.addKeyboardActionBinding("move-right", Keyboard.RIGHT);

You can add as many bindings to the same action as you'd like.

	binder.addKeyboardActionBinding("move-left", Keyboard.LEFT);
	binder.addKeyboardActionBinding("move-left", Keyboard.A);
	binder.addKeyboardActionBinding("move-right", Keyboard.RIGHT);
	binder.addKeyboardActionBinding("move-right", Keyboard.D);

### Gamepad bindings (buttons)
	
To add a gamepad binding, you use a similar syntax:

	binder.addGamepadActionBinding("move-left", GamepadControls.WINDOWS_DPAD_LEFT);
	binder.addGamepadActionBinding("move-right", GamepadControls.WINDOWS_DPAD_RIGHT);

To filter actions by player, you pass one additional parameter when adding the action.

	binder.addGamepadActionBinding("move-left-player-1", GamepadControls.WINDOWS_DPAD_LEFT, 0); // 0 = player 1

### Evaluating actions

Then, on your game loop block, you simply check whether any action is activated by using `isActionActivated()`:

	if (binder.isActionActivated("move-left")) {
		// Move the player to the left...
	} else if (binder.isActionActivated("move-right")) {
		// Move the player to the right...
	}

For actions that are not repeated, like a player jump, you can "consume" them via `consumeAction()`. This forces the player to activate the button again if they want to perform the action again.

	// During setup
	binder.addGamepadActionBinding("jump", GamepadControls.WINDOWS_BUTTON_ACTION_A);

	// During the loop
	if (isPlayerOnTheGround && binder.isActionActivated("jump")) {
		consumeAction("jump");
		// Perform jump...
	}

You can also check actions based on the time they were activated. This is especially useful for time-sensitive actions that are not verified all the time; otherwise, they could press the button way before an action was allowed to be performed - for example, a player pressing a button for "jump" while he/she is still in the air would jump immediately when touching the ground. To verify whether the player pressed jump in the past 0.3 seconds instead:

	// During setup
	binder.addGamepadActionBinding("jump", GamepadControls.WINDOWS_BUTTON_ACTION_A);

	// During the loop
	if (isPlayerOnTheGround && binder.isActionActivated("jump"), 0.03) {
		consumeAction("jump");
		// Perform jump...
	}

### Gamepad bindings (analog)

To handle sensitive gamepad controls, like axis or triggers, you create sensitive actions. Setup it first:

	binder.addGamepadSensitiveActionBinding("run-speed", GamepadControls.WINDOWS_L2_SENSITIVE); // L2/LT
	binder.addGamepadSensitiveActionBinding("axis-x", GamepadControls.WINDOWS_STICK_LEFT_X, NaN, -1, 1); // Any player, min value, max value
	binder.addGamepadSensitiveActionBinding("axis-y", GamepadControls.WINDOWS_STICK_LEFT_Y, NaN, -1, 1);

Then use it on your loop:

	var runSpeed:Number = binder.getActionValue("run-speed"); // Value will be between 0 and 
	var speedX:Number = binder.getActionValue("axis-x"); // Value will be between -1 and 1
	var speedY:Number = binder.getActionValue("axis-y");

### Stopping and resuming

By default, KeyActionBinder starts reading input events as soon as the instance is created. You can stop it with `stop()` and restart it with `start()`.

	// Stop interpreting input
	binder.stop();

	// Resume input
	binder.start();

### Different controls

As of now, `GamepadControls` has a list of known gamepad controls that you can add as action bindings to your game code. Notice that these controls are platform-dependent (e.g., `WINDOWS_DPAD_LEFT`, `OSX_DPAD_LEFT`, etc), This is likely to change in the future to allow an easier cross-platform setup via certain global controls.

### Events

If you'd rather use events (especially useful for user interfaces), KeyActionBinder also exposes control events for all actions.

	// Create input bindings
	binder.addKeyboardActionBinding("continue", Keyboard.ENTER);
	binder.addGamepadSensitiveActionBinding("trigger-press", GamepadControls.WINDOWS_L2_SENSITIVE);

	// Add callbacks to the event signals
	binder.onActionActivated.add(onActionActivated);
	binder.onActionDeactivated.add(onActionReleased);
	binder.onSensitiveActionChanged.add(onSensitiveActionChanged);

	private function onSensitiveActionChanged(__action:String, __value:Number):void {
		trace("The user activated the " + __action + " action's value. The new value is " + __value);
	}

	private function onActionActivated(__action:String):void {
		trace("The user activated the " + __action + " action by pressing a key or button.");
	}

	private function onActionDeactivated(__action:String):void {
		trace("The user deactivated the " + __action + " action by releasing a key or button.");
	}


## Other notes

 * If your GameInput controls suddenly stop working on OUYA or Android, this is likely due to a bug on Adobe AIR’s implementation. See [this post](http://zehfernando.com/2013/adobe-air-gameinput-pitfalls/) for reference and a workaround.

## Read more

 * Blog post: [Known OUYA GameInput controls on Adobe AIR](http://zehfernando.com/2013/known-ouya-gameinput-controls-on-adobe-air/) (July 2013)
 * Blog post: [Abstracting key and game controller inputs in Adobe AIR](http://zehfernando.com/2013/abstracting-key-and-game-controller-inputs-in-adobe-air/) (July 2013)
 * Blog post: [KeyActionBinder updates: time sensitive activations, new constants](http://zehfernando.com/2013/keyactionbinder-updates-time-sensitive-activations-new-constants/) (September 2013)

## To-do

 * Allow sensitive controls to be treated as normal controls (with a threshold)
 * Think of a way to avoid axis injecting button pressed
 * Add gamepad index to return signals, and rethink whether gamepad index should be part of isActionActivated() and getActionValue() instead
 * Use caching samples?
 * Allow "any" gamepad key (for "press any key")
 * Add missing asdocs
 * Better multi-platform setup... re-think GamepadControls