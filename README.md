# image-comparer

A Vue component for comparing 2 images with a slider

## Installation

```js
npm i @zweiav/image-comparer --registry=http://192.168.179.162:4873
```
```js
import ImageComparer from 'image-comparer';
```

## Usage

```html
<image-comparer before="/img/before.png" after="/img/after.png" />
```

## props
```js
{
    before: {
        type: String,
        required: true
    },
    beforeLabel: {
        type: String
    },
    after: {
        type: String,
        required: true
    },
    afterLabel: {
        type: String
    },
    offset: {
        type: [String, Number],
        default: 0.5,
        validator: (value) => {
            return (value > 0 && value <= 1)
        }
    }
}
```
