---
id: uncategorized-context-data
title: Context data
---

Some components might need to know a current form state. All this data is passed as `uniforms` in [React context](https://facebook.github.io/react/docs/context.html).

## Available context data

```js
MyComponentUsingUniformsContext.contextTypes = {
  uniforms: PropTypes.shape({
    name: PropTypes.arrayOf(PropTypes.string).isRequired,

    error: PropTypes.any,
    model: PropTypes.object.isRequired,

    schema: PropTypes.shape({
      getError: PropTypes.func.isRequired,
      getErrorMessage: PropTypes.func.isRequired,
      getErrorMessages: PropTypes.func.isRequired,
      getField: PropTypes.func.isRequired,
      getInitialValue: PropTypes.func.isRequired,
      getProps: PropTypes.func.isRequired,
      getSubfields: PropTypes.func.isRequired,
      getType: PropTypes.func.isRequired,
      getValidator: PropTypes.func.isRequired
    }).isRequired,

    state: PropTypes.shape({
      changed: PropTypes.bool.isRequired,
      changedMap: PropTypes.object.isRequired,
      submitting: PropTypes.bool.isRequired,

      label: PropTypes.bool.isRequired,
      disabled: PropTypes.bool.isRequired,
      placeholder: PropTypes.bool.isRequired
    }).isRequired,

    onChange: PropTypes.func.isRequired,
    onSubmit: PropTypes.func.isRequired,
    randomId: PropTypes.func.isRequired
  }).isRequired
};
```

## Example: `DisplayIf`

```js
import BaseField from 'uniforms/BaseField';
import nothing from 'uniforms/nothing';
import {Children} from 'react';

// We have to ensure that there's only one child, because returning an array
// from a component is prohibited.
const DisplayIf = ({children, condition}, {uniforms}) => (condition(uniforms) ? Children.only(children) : nothing);
DisplayIf.contextTypes = BaseField.contextTypes;

export default DisplayIf;
```

**Example:**

```js
const ThreeStepForm = ({schema}) => (
  <AutoForm schema={schema}>
    <TextField name="fieldA" />

    <DisplayIf condition={context => context.model.fieldA}>
      <section>
        <TextField name="fieldB" />

        <DisplayIf condition={context => context.model.fieldB}>
          <span>Well done!</span>
        </DisplayIf>
      </section>
    </DisplayIf>
  </AutoForm>
);
```

## Example: `SubmitButton`

```js
import BaseField from 'uniforms/BaseField';
import React from 'react';
import filterDOMProps from 'uniforms/filterDOMProps';

// This field works as follows: render standard submit field and disable it, when
// the form is invalid. It's a simplified version of a default SubmitField from
// uniforms-unstyled.
const SubmitField = (
  props,
  {
    uniforms: {
      error,
      state: {disabled, submitting, validating}
    }
  }
) => <input disabled={!!(error || disabled || submitting || validating)} type="submit" />;
SubmitField.contextTypes = BaseField.contextTypes;

export default SubmitField;
```

## Example: `SwapField`

```js
import BaseField from 'uniforms/BaseField';
import get from 'lodash/get';
import {Children} from 'react';
import {cloneElement} from 'react';

// This field works as follows: on click of its child it swaps values of fieldA
// and fieldB. It's that simple.
const SwapField = ({children, fieldA, fieldB}, {uniforms: {model, onChange}}) =>
  cloneElement(Children.only(children), {
    onClick() {
      const valueA = get(model, fieldA);
      const valueB = get(model, fieldB);

      onChange(fieldA, valueB);
      onChange(fieldB, valueA);
    }
  });
SwapField.contextTypes = BaseField.contextTypes;

export default SwapField;
```

**Example:**

```js
<section>
  <TextField name="firstName" />
  <SwapField fieldA="firstName" fieldB="lastName">
    <Icon name="refresh" />
  </SwapField>
  <TextField name="lastName" />
</section>
```