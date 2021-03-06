---
id: animations
title: Animations
layout: docs
category: Overview
permalink: docs/animations.html
next: accessibility
---

ReactXP supports a powerful animation abstraction. Individual style elements (e.g. transforms, opacity, or backgroundColor) can be animated.

## Animatable Components

Three base RX classes can have animatable styles:

* Animated.View

* Animated.Image

* Animated.Text

* Animated.TextInput

These component types should be used in place of the normal [View](components/view), [Image](components/image), [Text](components/text) or [TextInput](components/textinput) in the render method. In general, style properties expressed as numeric values or colors can be animated. Properties with text values (e.g. flexDirection or fontWeight) cannot be animated.

## Animated Values
The following example shows how to create animated values with an initial value. Animated values are typically stored as instance variables within a component class. They can also be stored in the state structure.

``` javascript
let animatedOpacityValue = RX.Animated.createValue(1.0);
let animatedScaleValue = RX.Animated.createValue(1.0);
```

For animated color values, it is possible to create interpolated values that map from a numeric range to a color range. In this example, the value smoothly transitions from white to red to black as the value increases from 0 to 1.

``` javascript
let animatedColorValue = RX.Animated.createValue(0.0);
let interpolatedValue = RX.Animated.interpolate(animatedColorValue,
    [0.0, 0.5, 1.0], ['white', 'red', 'black']);
```

## Animated Styles
Once an animated value is created, it can be associated with an animated style.

Some animated style values are more expensive than others. Some affect the layout of elements (e.g. width, height, top, left), so the layout engine needs to be invoked during each stage of the animations. It's faster to avoid these and stick to styles that don't affect the layout (e.g. opacity and transforms).

This example demonstrates how a style sheet can contain multiple animated values.
``` javascript
let animatedStyle = RX.Styles.createAnimatedViewStyle({
    opacity: animatedOpacityValue,
    transform: [{
        scale: animatedScaleValue
    }]
});
```

Animated style sheets can be combined with other static styles.
``` javascript
render() {
    <RX.Animated.View style={ [_styles.staticStyle, animatedStyle] } />
}
```

## Simple Timing Animations
To describe an animation, specify the final value of the animated value and a duration (specified in milliseconds). An optional easing function allows for a variety of animation curves including linear, step-wise, and cubic bezier.

Once an animation is defined, a call to the start() method starts the animation. The start method takes an optional parameter, a callback that is executed when the animation completes.

``` javascript
let opacityAnimation = RX.Animated.timing(animatedScaleValue,
    { toValue: 0.0, duration: 250, easing: RX.Animated.Easing.InOut() }
);

opacityAnimation.start(() => this._doSomethingWhenAnimationCompletes());
```

## Composite Animations
Sometimes it's useful to execute multiple animations in parallel or in sequence. This is easily accommodated by calling RX.Animated.parallel() or RX.Animated.sequence(). Composite animations can be nested to create sophisticated sequences.

``` javascript
let compositeAnimation = RX.Animated.parallel([
    RX.Animated.timing(animatedScaleValue,
        { toValue: 0.0, duration: 250, easing: RX.Animated.Easing.InOut() }
    ),
    RX.Animated.timing(animatedOpacityValue,
        { toValue: 1.1, duration 250, easing: RX.Animated.Easing.Linear() }
    )
]);

compositeAnimation.start();
```

## Directly Setting Animated Value
The value of an Animated Value can be set directly by calling the method ```setValue```. If this method is called while the value is being animated, the behavior is undefined. Setting the value of an Animated Value directly is faster than using a non-animated style attribute and re-rendering the component with a new attribute value.

``` javascript
let animatedOpacityValue = RX.Animated.createValue(1.0);

animatedOpacityValue.setValue(0.0);
```

## Web Limitations
ReactXP animation APIs on the web are implemented using DOM style transitions, as opposed to using JavaScript code to drive the animation. This results in much better performance and (in most cases) glitch-free animations, but it imposes some limitations on the use of the animation APIs.
* All active animated values associated with a particular element must share the same timing parameters (duration, easing function, delay, loop) and must be started at the same time.
* Each animated value can be associated with only one animated attribute that is actively running.
* Interpolated values used with startTransition are limited to only two values -- a begin and end value -- and must be specified in increasing order.
* Interpolated values not used with startTransition must have numeric outputValues, since we're interpolating between them ourselves.
* For interpolated values, the starting and ending values of a transition animation must correspond to the two interpolation keys.
* If an animation is stopped, the value will not reflect the intermediate position in the case of transforms and interpolated values.
