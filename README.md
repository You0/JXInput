JXInput
=======

Access XInput devices with Java code.

Visual Studio 2010 project contains the native code, which is already compiled and included in the Java project. It depends on XInput 1.3, which is used by Direct 9.0c games. XInput 1.4 is also supported through an extended API.

Requirements:

The [Visual C++ Redistributable Packages for Visual Studio 2013](https://www.microsoft.com/en-us/download/details.aspx?id=40784) is required to use JXInput. XInput 1.3 support comes out of the box in Windows 7. XInput 1.4 is only supported in Windows 8 or later.

Usage:

1.  Install the project into your Maven repository (since it is not available in the Central Maven repository).
2.  Include the Maven dependency into your project:

        <dependency>
            <groupId>com.ivan</groupId>
            <artifactId>xinput-device</artifactId>
            <version>0.3</version>
        </dependency>

3.  There is no need to extract or copy the native libraries from the VS project. The natives are included in the jar file and are extracted and loaded at runtime automatically under the `lib` folder of the working directory.
4.  In your Java code, import the `com.ivan.xinput.XInputDevice` class for XInput 1.3 functionality or `com.ivan.xinput.XInputDevice14` for XInput 1.4.
5.  To check if the desired version is available:

		// check if XInput 1.3 is available
		if (XInputDevice.isAvailable()) {
			System.out.println("XInput 1.3 is available on this platform.");
		}

		// check if XInput 1.4 is available
		if (XInputDevice14.isAvailable()) {
			System.out.println("XInput 1.4 is available on this platform.");
		}

6.  To retrieve devices:

        // retrieve all devices
        XInputDevice[] devices = XInputDevice.getAllDevices();
        
        // retrieve the device for player 1
        XInputDevice device = XInputDevice.getDeviceFor(0); // or devices[0]

        // use XInputDevice14 if you want to use the XInput 1.4 functions
        
7.  Using the device:

        XInputDevice device = ...;
        
        // first we need to poll data
        // will return false if the device is not connected
        if (device.poll()) {
            // retrieve the components
            XInputComponents components = device.getComponents();
            
            XInputButtons buttons = components.getButtons();
            XInputAxes axes = components.getAxes();
            
            // buttons and axes have public fields
            
            // retrieve button state
            if (button.a) {
                // button A is currently pressed
            }
            
            // retrieve axis state
            float acceleration = axes.rt;
            float brake = axes.lt;
        } else {
            // controller is not connected; display a message
        }
        
        // this is exactly the same as above
        device.poll();
        if (device.isConnected()) {
            // ...
        } else {
            // ...
        }

8.  Using deltas (changes in state):

        XInputDevice device = ...;
        if (device.poll()) {
            // retrieve the delta
            XInputComponentsDelta delta = device.getDelta();
            
            XInputButtonsDelta buttons = delta.getButtons();
            XInputAxesDelta axes = delta.getAxes();
            
            // retrieve button state change
            if (buttons.isPressed(XInputButton.a)) {
                // button A was just pressed
            } else if (buttons.isReleased(XInputButton.a)) {
                // button A was just released
            }
            
            // retrieve axis state change
            // the class provides methods for each axis
            // and a method for providing an XInputAxis
            float accelerationDelta = axes.getRTDelta();
            float brakeDelta = axes.getDelta(XInputAxis.leftTrigger);
        } else {
            // controller is not connected; display a message
        }

9.  Using a listener:

        XInputDevice device = ...;
        
        // the SimpleXInputDeviceListener allows us to implement only the methods
        // we actually need
        XInputDeviceListener listener = new SimpleXInputDeviceListener() {
            @Override
            public void connected() {
                // resume the game
            }
            
            @Override
            public void disconnected() {
                // pause the game and display a message
            }
            
            @Override
            public void buttonChanged(final XInputButton button, final boolean pressed) {
                // the given button was just pressed (if pressed == true) or released (pressed == false)
            }
            
            @Override
            public void axisChanged(final XInputAxis axis, final float value, final float delta) {
                // the given axis has changed to value by delta
            }
        };
        
        // whenever the device is polled, listener events will be fired as long as there are changes
        device.poll();

10.  XInput 1.4 specific (UNTESTED!)

	a.  Enable or disable the XInput reporting state

		// use this when your application loses focus
		XInputDevice14.setEnabled(false);
		// - polling will return neutral data regardless of actual state (e.g. sticks at rest, buttons released)
		// - vibration settings will be ignored
		
		// use this when your application regains focus
		XInputDevice14.setEnabled(true);
		// - polling will return the actual state of the controller
		// - vibration settings will be applied
		
	b.  Retrieve battery information from a device

		XInputDevice14 device = ...;

		// retrieves the gamepad battery data
        XInputBatteryInformation gamepadBattInfo = device.getBatteryInformation(XInputBatteryDeviceType.GAMEPAD);
        System.out.println("Gamepad battery: " + gamepadBattInfo.getType() +", " + gamepadBattInfo.getLevel());

        // check battery level
        if (gamepadBattInfo.getLevel() == XInputBatteryLevel.LOW) {
            System.out.println("  Battery is low! Recharge or replace batteries.");
        }

    c.  Retrieve device capabilities

    	XInputDevice14 device = ...;

        XInputCapabilities caps = device.getCapabilities();
        System.out.println("Device type: " + caps.getType());
        System.out.println("Device subtype: " + caps.getSubType());

        // caps.getSupportedButtons() returns a Set<XInputButton> with the supported buttons
        // caps.getResolutions() returns an object with the resolutions of all axes

    d.  Retrieve a keystroke

    	XInputDevice14 device = ...;

    	XInputKeystroke keystroke = device.getKeystroke();
    	// use keystroke.isKeyDown(), .isKeyUp() and .isRepeat() to check the kind of keystroke
    	// use keystroke.getVirtualKey() to get the virtual key code (constants available in XInputVirtualKeyCodes)
    	// use keystroke.getUnicode() to get the Unicode character

Released under the [MIT License](http://opensource.org/licenses/MIT).
