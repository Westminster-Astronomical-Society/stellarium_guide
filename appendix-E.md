# Appendix E: Script Snippets

Here you will find various script snippets that can be used within Stellarium to
automate tasks, customize views, or perform specific tasks.

## Utility Functions

### Pause Script Execution

There is no option in Stellarium to pause execution of a script directly. The
following is a work around that puts the script in a loop until the user presses
the 'L' key. It works by setting the time rate to a very low value and waiting
until the rate changes (user presses the 'L' key). There are a number of ways to
achieve the same but this produces no perceptible change in the display and also
effectively pauses the passage of time. If you want to keep time running you can use
the `fastForward()` function below with `wait='L'` instead.

```javascript
// a function to pause the script until the user presses the 'L' key
function pauseKey(label = true) {
    var timerate = core.getTimeRate();
    core.setTimeRate(0.05);
    L = LabelMgr.labelScreen("Paused ('L' to continue)", 20, core.getScreenHeight() - 60,
        visible = false, fontSize = 18, fontColor = "#666666");
    if (label) {
        LabelMgr.setLabelShow(L, true);
    }
    do {
        core.wait(1);
    }
    while (core.getTimeRate() < 0.1);
    core.setTimeRate(timerate);
    LabelMgr.deleteLabel(L);
};
```

```{note}
There is now a pause function available in core module called `waitForKeypress()`. This can be used as a simpler alternative to the `pauseKey()` function above. It takes a label string as an argument to override the default message ("Press key to continue..."). An empty string will remove the message entirely.
```


### Fast Forward Time

This function allows you to fast forward time at a specified rate, wait until a
specified date/time is reached and restore the previous rate. The default time
`spec` is `"local"` but can be set to `"utc"` if you like. If no time is
specified the default `wait` parameter is set to `"L"` to wait for the user to
press the 'L' key.

```javascript
// Fast forwards time at a specified rate and until it reaches the provided time.
function fastForward(rate, wait = "L", spec = "local") {
    var timerate = core.getTimeRate();
    core.setTimeRate(rate);
    if (wait == "L") {
        L = LabelMgr.labelScreen("Rate = " + rate + (" press 'L' to resume"), 20, core.getScreenHeight() - 60,
            visible = true, fontSize = 18, fontColor = "#666666");
        i = 0;
        do {
            core.wait(1);
            i++;
        }
        while (core.getTimeRate() <= rate);
        core.setTimeRate(timerate);
        LabelMgr.deleteLabel(L);
    }
    else {
        core.waitFor(wait, spec = spec);
        core.setTimeRate(timerate);
    }
};
```

### Set Rise and Set Times

The following function allows you to set the time to the rise, set or transit
times of a specified celestial body. It uses the time string returned by the
`core.getRTS()` function to retrieve the rise, set, and transit times of the
specified celestial body and set the time accordingly.

```javascript
function setTimeFromRTS(timeString)
{
    var date = core.getDate("local");      // "2026-06-29T14:23:51"
    date = date.substring(0, 10);          // "2026-06-29"

    // Convert "20h39m" -> "20:39:00"
    var time = timeString.replace("h", ":").replace("m", ":00");

    core.setDate(date + "T" + time, "local");
}
```

You can use this to set the time to sunrise, sunset.

```javascript
function setSunrise()
{
    var rts = core.getRTS("Sun");
    setTimeFromRTS(rts["rise"]);
}

function setSunset()
{
    var rts = core.getRTS("Sun");
    setTimeFromRTS(rts["set"]);
}
```

You can specify an altitude to calculate different twilight times. For example,
astronomical twilight occurs when the Sun is 18 degrees below the horizon.

```javascript
function setAstTwilight(which)
{
    var rts = core.getRTS("Sun", -18);
    if (which === "start") {            //morning
        setTimeFromRTS(rts["rise"]);
    } else if (which === "end") {       //evening
        setTimeFromRTS(rts["set"]);
    }
}
```

The `core.getRTS()` function can only be used to get the rise, set, and transit
times for the current date, and local time. There is a `solar_calc.js` script in
the [Stellarium Template](https://github.com/Westminster-Astronomical-Society/Stellarium_Template)
repository on GitHub that you can use to get sunrise, sunset, and twilight times
for other dates.


## Images

### Load a Custom Sky Image

The following function simplifies loading a custom sky image into Stellarium.
For this function you should use a **square** image with background set to black
(black is transparent). The `params` option is a JavaScript object containing
the necessary parameters for displaying the image. You can then call
`loadImage(alf_CMa)` to load this specific image. Images can be preloaded with
`visible` set to `false` and displayed when needed.

```javascript
// Definition of the alf_CMa sky image object
alf_CMa = {
    id: "alf_CMa",
    path: "../images/alf_CMa.png",
    ra: "6h 45m 08.0s",
    dec: "-16d 43m 30.0s",
    angSize: 1.5, // arcminutes
    rotation: 0,  // degrees
    minRes: 2.5,
    maxBright: 14,
    visible: false
};

// Loads a custom sky image into Stellarium
function loadImage(params) {
    core.loadSkyImage(
        params.id,
        params.path,
        params.ra,
        params.dec,
        params.angSize,
        params.rotation,
        params.minRes,
        params.maxBright,
        params.visible
    );
};
```

### Delete all Sky Images

There is no built-in function to delete all sky images at once, but you can
achieve this by iterating over all loaded images and deleting them individually
using `core.removeSkyImage(id)`. The function below gets the IDs of all loaded sky
images and deletes them, skipping the first two, leaving the default images
intact.

```javascript
// Deletes all sky images currently loaded in Stellarium except defaults.
function deleteAllSkyImages() {
    var keys = StelSkyLayerMgr.getAllKeys();

    if (keys.length > 2) {
        for (var i = 2; i < keys.length; i++) {
            core.removeSkyImage(keys[i]);
        };
    };
};
```

