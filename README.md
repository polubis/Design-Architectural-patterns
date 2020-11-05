# Design-Architectural-patterns

## Some good rules / principles

### DRY - Don't Repeat Yourself

*"DRY is a guideline, not a religion. Those who take it to the point of DRY above all else, have taken it too far."*

Repeating the same code in the application may considerably take longer to make the necessary changes and may cause bugs - but not always.

#### Styles

> BAD - duplication in css causes issues with changing layout - designer change his mind
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

> NOT PERFECT BUT STILL OK - duplication only in two files
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

> OK - mixins used to avoid any code duplication - we can compose our css rules
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

#### Constant files

> BAD - helper function `makePaths` makes code hard to read - also if someone change implementation - app destroyed
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

> NOT PERFECT BUT STILL OK - duplication easy to change via code editor
```ts
const FORGOTTEN_PASSWORD = 'Account/ForgottenPassword';
const REGISTER = 'Account/Register';
const GET_SELF = 'Account/GetCurrentUserData';
```

> OK - no duplication, easy code
```ts
const CONTROLLER = 'Account';

const FORGOTTEN_PASSWORD = `${CONTROLLER}/ForgottenPassword`;
const REGISTER = `${CONTROLLER}/Register`;
const GET_SELF = `${CONTROLLER}/GetCurrentUserData`;
```

#### React component guards

> BAD - check user admin / rendering children features are duplicated / hard to read nested ternary operator
```ts
const Admin = ({ children }: Guard.Props) => {
  const { pending, authorized, ...state } = useAuthProvider();

  return pending
    ? null
    : authorized
    ? state.user.roles.includes(UserRole.Admin)
      ? typeof children === 'function'
        ? children(state)
        : children
      : null
    : null;
};

const Protected = ({ children }: Guard.Props) => {
  const { pending, authorized, ...state } = useAuthProvider();

  return pending
    ? null
    : authorized
    ? typeof children === 'function'
      ? children(state)
      : children
    : null;
};

const Unprotected = ({ children }: Guard.Props) => {
  const { pending, authorized, ...state } = useAuthProvider();

  return pending
    ? null
    : authorized
    ? null
    : typeof children === 'function'
    ? children(state)
    : children;
};

// and 5 more guard types 
```

> NOT PERFECT BUT STILL OK - some code duplication / but only in two places so who cares
```ts
const Admin = ({ children }: Guard.Props) => {
  const { pending, authorized, ...state } = useAuthProvider();

  return pending
    ? null
    : authorized
    ? state.user.roles.includes(UserRole.Admin)
      ? typeof children === 'function'
        ? children(state)
        : children
      : null
    : null;
};

const Protected = ({ children }: Guard.Props) => {
  const { pending, authorized, ...state } = useAuthProvider();

  return pending
    ? null
    : authorized
    ? typeof children === 'function'
      ? children(state)
      : children
    : null;
};
```

> OK - 2 helpers functions created / guards code easier to read / redundant logic is implemented in helper functions
```ts
const isAdmin = (roles: string[]): boolean => roles.includes(UserRole.Admin);

const renderChildren = (
  children: JSX.Element | Guard.Children.RenderProp,
  state: AuthProvider.State
): JSX.Element => (typeof children === 'function' ? children(state) : children);

const Admin = ({ children }: Guard.Props) => {
  const { pending, authorized, ...state } = useAuthProvider();
  
  return pending
    ? null
    : authorized
    ? isAdmin(state.user.roles)
      ? renderChildren(children, state)
      : null
    : null;
};

const Protected = ({ children }: Guard.Props) => {
  const { pending, authorized, ...state } = useAuthProvider();

  return pending ? null : authorized ? renderChildren(children, state) : null;
};

const Unprotected = ({ children }: Guard.Props) => {
  const { pending, authorized, ...state } = useAuthProvider();

  return pending ? null : authorized ? null : renderChildren(children, state);
};
```

### KISS

### Code Against Interfaces, Not Implementations

### SOLID

### Composition over inheritance
