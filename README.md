# image-comparer

A Vue component for comparing 2 images with a slider

## Installation

```bash
yarn add @twoavy/image-comparer 
```

```js
import ImageComparer from '@twoavy/image-comparer'
```

## Usage

```html
<image-comparer before="/img/before.png" after="/img/after.png" />
```

### Props

Name | Type | Default | Note
:--- | :---: | :---: | ---
before | string | required | upper/left image
after | string | required | lower/right image
afterLabel | string | |
afterLabel | string | |
offset | number | 0.5 | slider start pos
clickToMove | boolean | true | move only on handle
handleWidth | number | 60/30 | (horizontal/vertical)
handleHeight | number | 30/60 | (horizontal/vertical)
orientation | boolean | horizontal

### Events


##### @update
fired when slider moves. returns slider position
```
{ x: 0.2321, y: 0.9342 } // 0 <= value <= 1
```
