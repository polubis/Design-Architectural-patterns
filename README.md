# Design-Architectural-patterns

## Some good rules / principles

### DRY - Don't Repeat Yourself

"DRY is a guideline, not a religion. Those who take it to the point of DRY above all else, have taken it too far."

Repeating the same code in the application may considerably take longer to make the necessary changes and may cause bugs.

> DRY FRIENDLY
```scss
@mixin text {
  font-family: $fontPrimary;
  text-transform: none;
}

@mixin label {
  @include text;
  font-size: 14px;
  font-weight: $medium;
  line-height: 24px;
}

// in modal.scss
h2 {
  @include label;
}

h2 {
  @include label;
} // in header.scss
```

> NOT PERFECT BUT STILL OK
```scss
// in modal.scss
h2 {
  font-family: $fontPrimary;
  text-transform: none;
  font-size: 14px;
  font-weight: $medium;
  line-height: 24px;
}

// in header.scss
h2 {
  font-family: $fontPrimary;
  text-transform: none;
  font-size: 14px;
  font-weight: $medium;
  line-height: 24px;
} 
```

> BAD
```scss
// in modal.scss
h2 {
  font-family: $fontPrimary;
  text-transform: none;
  font-size: 14px;
  font-weight: $medium;
  line-height: 24px;
}

// in header.scss
h2 {
  font-family: $fontPrimary;
  text-transform: none;
  font-size: 14px;
  font-weight: $medium;
  line-height: 24px;
}

// ...
// in button.scss
// in prompt.scss
// in input.scss
// in textarea.scss
// in datepicker.scss
```

> DRY

```ts
const CONTROLLER = 'Account';

const FORGOTTEN_PASSWORD = `${CONTROLLER}/ForgottenPassword`;
const REGISTER = `${CONTROLLER}/Register`;
const GET_SELF = `${CONTROLLER}/GetCurrentUserData`;
```

> NOT PERFECT BUT STILL OK

```ts
const FORGOTTEN_PASSWORD = 'Account/ForgottenPassword';
const REGISTER = 'Account/Register';
const GET_SELF = 'Account/GetCurrentUserData';
```

> BAD - code is complicated

```ts
export const [FORGOTTEN_PASSWORD, REGISTER, GET_SELF] = makePaths('Account')(
  'ForgottenPassword',
  'Register',
  'GetCurrentUserData'
);

export const [GET_PATTERNS, EDIT_PATTERN, ADD_PATTERN, GET_PATTERN, DELETE_PATTERN] = makePaths(
  'TemplatePatterns'
)('Search', 'Update', 'Add', 'Get', 'Delete');
```


### KISS

### Code Against Interfaces, Not Implementations

### SOLID

### Composition over inheritance
