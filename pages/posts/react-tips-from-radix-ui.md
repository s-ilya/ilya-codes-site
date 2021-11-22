---
layout: base.njk
title: React tips from Radix UI
date: 2021-11-22
tags: post
---
# {{title}}
Some tips for using React I learned going thought the amazing [Radix UI library](https://www.radix-ui.com/)

## Export for both named and * as imports
Consider two usages of React components (or anything else actually):

```jsx
import { Checkbox, CheckboxIndicator } from './Checkbox';

<Checkbox>
  <CheckboxIndicator />
</Checkbox>
```
```jsx
import * as Checkbox from '@radix-ui/react-checkbox';

<Checkbox.Root>
  <Checkbox.Indicator />
</Checkbox.Root>
);
```
The neat trick is to have suitable export names for both cases
```jsx
const Root = Checkbox;
const Indicator = CheckboxIndicator;

export {
  Checkbox,
  CheckboxIndicator,
  //
  Root,
  Indicator,
};
```