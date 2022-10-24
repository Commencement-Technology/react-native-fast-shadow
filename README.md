# react-native-fast-shadow

🌖 **Fast** and **high quality** Android shadows for React Native

## Why

React Native only supports shadows on Android through the [elevation](https://reactnative.dev/docs/view-style-props#elevation-android) prop but achieving the effect you want is often impossible as it only comes with very few presets. Third-party libraries have been created to circumvent this but when used on many views, they can make you app slower or significantly increase the memory consumption of your app.

## Features
* 💆‍♀️ **Easy to use:** Drop-in replacement for the `<View>` component
* 🎛 **Customizable:** Supports all the regular shadow props: `shadowRadius`, `shadowColor`, `shadowOpacity` and `shadowOffset`
* ⚡️ **Performant:** Shadows can be applied to a large number of views without any signicant performance impact. It is optimized for low memory consumption and fast rendering

## Getting started

```sh
npm install react-native-fast-shadow
or
yarn add react-native-fast-shadow
```

**Usage:**

```jsx
import { ShadowedView } from "react-native-fast-shadow";

<ShadowedView
  style={{
    shadowColor: 'black',
    shadowOpacity: 0.4,
    shadowRadius: 12,
    shadowOffset: {
      width: 5,
      height: 3,
    },
  }}
>
  <Image source={require('./kitten.png')} style={{ borderRadius: 30 }} />
</ShadowedView>
```

<img width="198" src="https://user-images.githubusercontent.com/20420653/197513322-81c46d07-2f44-463b-86ef-86a4ad856146.png">

## How it works under the hood

On Android, shadow drawables are generated with the following process (see [ShadowFactory.java](https://github.com/alan-eu/react-native-fast-shadow/blob/main/android/src/main/java/com/reactnativefastshadow/ShadowFactory.java) for more details):
1. The shape of the view (rectangle with rounder corners) is painted in black on a canvas
2. A gaussian blur is applied to the canvas with the given `shadowRadius` using the Renderscript API
3. The drawable is then converted to a [NinePatchDrawable](https://developer.android.com/reference/android/graphics/drawable/NinePatchDrawable) to ensure that corners of the shadow won't be distorted when the view is resized. This way, we can generate only a small shadow drawable and **reuse it** for all views with the same border and blur radii.
4. Finally, the drawable is rendered behind the view content: it is tinted with `shadowColor`/`shadowOpacity` and offseted according to `shadowOffset` 

**How NinePatchDrawable works** (notice how the corners are not streched when the drawable is resized):

<img width="240" src="https://user-images.githubusercontent.com/20420653/197518195-2e13d80e-2a24-4e1c-ae53-444306733c83.gif">

**Note:** on other platforms (iOS or web), `<ShadowedView>` is just an alias of React Native's `<View>` as these platforms already support shadows out of the box.

## Limitations

React-native-fast-shadow comes with some limitations:
* **It only works with rounded rectangles:** Unlike the iOS `<View>` implementation, `<ShadowedView>` won't work with freeform views. It expects its descendant views to be a rounded rectangle. **Solutions:** For `<Text>` elements, you can use [textShadowRadius](https://reactnative.dev/docs/text-style-props.html#textshadowradius). For complex shapes, [react-native-androw](https://github.com/folofse/androw) can be used.
* **\<ShadowedView\> expects its child view to fill it:** It's up to you to make sure that `<ShadowedView>` and its children have the same size, otherwise the shadow will be larger than the content.
* **Shadow radius is limited to 25 or below:** This limitation comes from Renderscript's [blur effect](https://developer.android.com/reference/android/renderscript/ScriptIntrinsicBlur). **Solution:** Let us know if this is an issue for you. This can probably be worked around by downscaling the shadow bitmap before applying the blur effect

## Benchmark

