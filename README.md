# Advanced transforms

## Current state

Currently, CSS only supports affine transformations like translation, rotation, etc.
Those are all transformations that can be expressed by a 3x3 matrix.

Transformations beyond that are not possible with CSS, at the moment.

## Use cases

There are a bunch of use cases for more advanced transformations. Here are some examples:

* Bending: https://github.com/w3c/csswg-drafts/issues/6293, https://github.com/w3c/csswg-drafts/issues/4845
* Twisting: https://github.com/w3c/csswg-drafts/issues/4845
* Rhombus: https://stackoverflow.com/questions/53917297/css-rhombus-with-angles-120-60-120-60, https://codepen.io/Mihnutzen/pen/JjyJVB

## History

Back in 2011, with [CSS Shaders](https://drafts.fxtf.org/custom/) aka Custom Filters there was an approach to bring advanced transforming and image manipulation effects to CSS. It was relying on vertex and pixel shaders, which could be applied to elements via the `filter` property.

It ultimately failed because it was too complex, both for implementers as well as CSS authors. And there were also a bunch of [security concerns](https://www.w3.org/Graphics/fx/wiki/CSS_Shaders_Security) raised.

## Idea

Here's an idea that covers the advanced transforms part. It's a different, more CSSy approach.

The idea is to introduce a way to create grids of vertices on an element and move those vertices around. It is based on the [vertex mesh described in CSS Shaders](https://drafts.fxtf.org/custom/#vertex-mesh) and is also something that many image manipulation tools provide.

## Proposal

The concrete proposal is to add a `vertices-translate()` function to the `transform` property. That function takes a bunch of x- and y-coordinates representing the deltas of the original positions of the vertices.

These coordinates are defined row-wise from left to right and top to bottom.

The simplest vertex mesh is described by the four corner points of an element.

With that, you can express the different affine transformations we already have.

### Examples for affine transformations

#### Identity

```css
.identity {
  transform: vertices-translate(0 0, 0 0 / 0 0, 0 0);
}
```

#### Translation

Moving an element 20 pixels down.

```css
.translation {
  transform: vertices-translate(0 20px, 0 20px / 0 20px, 0 20px);
}
```

#### Reflection

Horizontally reflecting an element.

```css
.reflection {
  transform: vertices-translate(100% 0, -100% 0 / 100% 0, -100% 0);
}
```

#### Scale

Making an element double its size.

```css
.scale {
  transform: vertices-translate(-50% -50%, 50% -50% / -50% 50%, 50% 50%);
}
```

#### Rotation

Rotating an element 45 degrees clockwise.

```css
.rotation {
  --angle: 45deg;
  transform: vertices-translate(
    calc(cos(var(--angle)) * -50% - sin(var(--angle)) * -50%) calc(sin(var(--angle)) * -50% + cos(var(--angle)) * -50%),
    calc(cos(var(--angle)) * 50% - sin(var(--angle)) * -50%) calc(sin(var(--angle)) * 50% + cos(var(--angle)) * -50%) /
    calc(cos(var(--angle)) * -50% - sin(var(--angle)) * 50%) calc(sin(var(--angle)) * -50% + cos(var(--angle)) * 50%),
    calc(cos(var(--angle)) * 50% - sin(var(--angle)) * 50%) calc(sin(var(--angle)) * 50% + cos(var(--angle)) * 50%)
  );
}
```

#### Skew

Skewing an element 30 degrees.

```css
.skew {
  --angle: 30deg;
  transform: vertices-translate(
    calc(100% + tan(var(--angle)) * -50%) 0,
    calc(100% + tan(var(--angle)) * 50%) 0 /
    calc(100% + tan(var(--angle)) * -50%) 0,
    calc(100% + tan(var(--angle)) * 50%) 0
  );
}
```

### Examples for non-affine transformations

#### Trapezoid

```css
.trapezoid {
  transform: vertices-translate(10% 0, -10% 0 / -10% 0, 10% 0);
}
```

#### Rhombus

```css
.rhombus {
  transform: vertices-translate(
    50% 0, -50% 0 /
    0 0, 0 0 /
    50% 0, -50% 0);
}
```

#### Animating the vertices

This example shows how you can create a simple wave animation using a grid of 4x2 vertices.

```css
@property --vertex-1-angle {
  syntax: '<angle>';
  inherits: false;
  initial-value: 0deg;
}

@property --vertex-2-angle {
  syntax: '<angle>';
  inherits: false;
  initial-value: 0deg;
}

@property --vertex-3-angle {
  syntax: '<angle>';
  inherits: false;
  initial-value: 0deg;
}

@property --vertex-4-angle {
  syntax: '<angle>';
  inherits: false;
  initial-value: 0deg;
}

@keyframes wave {
  from {
    --vertex-1-angle: 0deg;
    --vertex-2-angle: 90deg;
    --vertex-3-angle: 180deg;
    --vertex-4-angle: 270deg;
  }

  to {
    --vertex-1-angle: 360deg;
    --vertex-2-angle: 450deg;
    --vertex-3-angle: 540deg;
    --vertex-4-angle: 630deg;
  }
}

.wave {
  --amplitude: 50px;
  transform: vertices-translate(
    0 calc(sin(var(--vertex-1-angle)) * var(--amplitude)), 0 calc(sin(var(--vertex-2-angle)) * var(--amplitude)), 0 calc(sin(--vertex-3-angle) * var(--amplitude)), 0 calc(sin(var(--vertex-4-angle)) * var(--amplitude)) /
    0 calc(sin(var(--vertex-1-angle)) * var(--amplitude)), 0 calc(sin(var(--vertex-2-angle)) * var(--amplitude)), 0 calc(sin(--vertex-3-angle) * var(--amplitude)), 0 calc(sin(var(--vertex-4-angle)) * var(--amplitude))
  );
  animation: wave linear infinite;
}
```